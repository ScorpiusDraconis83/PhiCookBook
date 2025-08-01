<!--
CO_OP_TRANSLATOR_METADATA:
{
  "original_hash": "9a626d7522772d8b7b6f188dc79108c4",
  "translation_date": "2025-07-16T20:26:26+00:00",
  "source_file": "md/01.Introduction/03/iOS_Inference_MLX.md",
  "language_code": "es"
}
-->
# Ejecutando Phi-3 y Phi-4 en iOS con el Framework Apple MLX

Este tutorial muestra cómo crear una aplicación iOS que ejecute el modelo Phi-3 o Phi-4 directamente en el dispositivo, usando el framework Apple MLX. [MLX](https://opensource.apple.com/projects/mlx/) es el framework de aprendizaje automático de Apple optimizado para chips Apple Silicon.

## Requisitos previos

- macOS con Xcode 16 (o superior)
- Dispositivo iOS 18 (o superior) con al menos 8GB (iPhone o iPad compatible con los requisitos de Apple Intelligence, ya que estos son similares a los requisitos cuantificados de Phi)
- conocimientos básicos de Swift y SwiftUI

## Paso 1: Crear un nuevo proyecto iOS

Comienza creando un nuevo proyecto iOS en Xcode:

1. abre Xcode y selecciona "Create a new Xcode project"
2. elige "App" como plantilla
3. nombra tu proyecto (por ejemplo, "Phi3-iOS-App") y selecciona SwiftUI como interfaz
4. elige una ubicación para guardar tu proyecto

## Paso 2: Añadir las dependencias necesarias

Agrega el paquete [MLX Examples](https://github.com/ml-explore/mlx-swift-examples) que contiene todas las dependencias y utilidades necesarias para precargar modelos y realizar inferencias:

```swift
// In Xcode: File > Add Package Dependencies
// URL: https://github.com/ml-explore/mlx-swift-examples
```

Aunque el paquete base [MLX Swift](https://github.com/ml-explore/mlx-swift) es suficiente para operaciones tensoriales básicas y funcionalidades ML esenciales, el paquete MLX Examples ofrece varios componentes adicionales diseñados para trabajar con modelos de lenguaje y facilitar el proceso de inferencia:

- utilidades para cargar modelos que manejan la descarga desde Hugging Face
- integración del tokenizador
- helpers para generación de texto durante la inferencia
- definiciones de modelos preconfiguradas

## Paso 3: Configurar los permisos (Entitlements)

Para permitir que nuestra app descargue modelos y asigne suficiente memoria, necesitamos añadir permisos específicos. Crea un archivo `.entitlements` para tu app con el siguiente contenido:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>com.apple.security.app-sandbox</key>
    <true/>
    <key>com.apple.security.files.user-selected.read-only</key>
    <true/>
    <key>com.apple.security.network.client</key>
    <true/>
    <key>com.apple.developer.kernel.increased-memory-limit</key>
    <true/>
</dict>
</plist>
```

> **Note:** El permiso `com.apple.developer.kernel.increased-memory-limit` es importante para ejecutar modelos más grandes, ya que permite que la app solicite más memoria de la habitual.

## Paso 4: Crear el modelo de mensaje de chat

Primero, vamos a crear una estructura básica para representar nuestros mensajes de chat:

```swift
import SwiftUI

enum MessageState {
    case ok
    case waiting
}

struct ChatMessage: Identifiable {
    let id = UUID()
    let text: String
    let isUser: Bool
    let state: MessageState
}
```

## Paso 5: Implementar el ViewModel

A continuación, crearemos la clase `PhiViewModel` que se encarga de cargar el modelo y realizar la inferencia:

```swift
import MLX
import MLXLLM
import MLXLMCommon
import SwiftUI

@MainActor
class PhiViewModel: ObservableObject {
    @Published var isLoading: Bool = false
    @Published var isLoadingEngine: Bool = false
    @Published var messages: [ChatMessage] = []
    @Published var prompt: String = ""
    @Published var isReady: Bool = false
    
    private let maxTokens = 1024
    
    private var modelContainer: ModelContainer?
    
    func loadModel() async {
        DispatchQueue.main.async {
            self.isLoadingEngine = true
        }
        
        do {
            MLX.GPU.set(cacheLimit: 20 * 1024 * 1024)
            
            // Phi 3.5 mini is preconfigured in Swift MLX Examples
            let modelConfig = ModelRegistry.phi3_5_4bit
            
            // Phi 4 mini can be pulled from Hugging Face, but requires referencing Swift MLX Examples from the main branch
            //let modelConfig = ModelConfiguration(
            //    id: "mlx-community/Phi-4-mini-instruct-4bit",
            //    defaultPrompt: "You are a helpful assistant.",
            //    extraEOSTokens: ["<|end|>"]
            //)
            
            print("Loading \(modelConfig.name)...")
            self.modelContainer = try await LLMModelFactory.shared.loadContainer(
                configuration: modelConfig
            ) { progress in
                print("Download progress: \(Int(progress.fractionCompleted * 100))%")
            }
            
            // Log model parameters
            if let container = self.modelContainer {
                let numParams = await container.perform { context in
                    context.model.numParameters()
                }
                print("Model loaded. Parameters: \(numParams / (1024*1024))M")
            }
            
            DispatchQueue.main.async {
                self.isLoadingEngine = false
                self.isReady = true
            }
        } catch {
            print("Failed to load model: \(error)")
            
            DispatchQueue.main.async {
                self.isLoadingEngine = false
            }
        }
    }
    
    func fetchAIResponse() async {
        guard !isLoading, let container = self.modelContainer else {
            print("Cannot generate: model not loaded or already processing")
            return
        }
        
        let userQuestion = prompt
        let currentMessages = self.messages
        
        DispatchQueue.main.async {
            self.isLoading = true
            self.prompt = ""
            self.messages.append(ChatMessage(text: userQuestion, isUser: true, state: .ok))
            self.messages.append(ChatMessage(text: "", isUser: false, state: .waiting))
        }
        
        do {
            let _ = try await container.perform { context in
                var messageHistory: [[String: String]] = [
                    ["role": "system", "content": "You are a helpful assistant."]
                ]
                
                for message in currentMessages {
                    let role = message.isUser ? "user" : "assistant"
                    messageHistory.append(["role": role, "content": message.text])
                }
                
                messageHistory.append(["role": "user", "content": userQuestion])
                
                let input = try await context.processor.prepare(
                    input: .init(messages: messageHistory))
                let startTime = Date()
                
                let result = try MLXLMCommon.generate(
                    input: input,
                    parameters: GenerateParameters(temperature: 0.6),
                    context: context
                ) { tokens in
                    let output = context.tokenizer.decode(tokens: tokens)
                    Task { @MainActor in
                        if let index = self.messages.lastIndex(where: { !$0.isUser }) {
                            self.messages[index] = ChatMessage(
                                text: output,
                                isUser: false,
                                state: .ok
                            )
                        }
                    }
                    
                    if tokens.count >= self.maxTokens {
                        return .stop
                    } else {
                        return .more
                    }
                }
                
                let finalOutput = context.tokenizer.decode(tokens: result.tokens)
                Task { @MainActor in
                    if let index = self.messages.lastIndex(where: { !$0.isUser }) {
                        self.messages[index] = ChatMessage(
                            text: finalOutput,
                            isUser: false,
                            state: .ok
                        )
                    }
                    
                    self.isLoading = false
                    
                    print("Inference complete:")
                    print("Tokens: \(result.tokens.count)")
                    print("Tokens/second: \(result.tokensPerSecond)")
                    print("Time: \(Date().timeIntervalSince(startTime))s")
                }
                
                return result
            }
        } catch {
            print("Inference error: \(error)")
            
            DispatchQueue.main.async {
                if let index = self.messages.lastIndex(where: { !$0.isUser }) {
                    self.messages[index] = ChatMessage(
                        text: "Sorry, an error occurred: \(error.localizedDescription)",
                        isUser: false,
                        state: .ok
                    )
                }
                self.isLoading = false
            }
        }
    }
}

```

El ViewModel muestra los puntos clave de integración con MLX:

- establecer límites de caché GPU con `MLX.GPU.set(cacheLimit:)` para optimizar el uso de memoria en dispositivos móviles
- usar `LLMModelFactory` para descargar el modelo bajo demanda e inicializar el modelo optimizado para MLX
- acceder a los parámetros y estructura del modelo a través de `ModelContainer`
- aprovechar la generación token a token de MLX mediante el método `MLXLMCommon.generate`
- gestionar el proceso de inferencia con configuraciones adecuadas de temperatura y límites de tokens

El enfoque de generación de tokens en streaming ofrece retroalimentación inmediata al usuario mientras el modelo genera texto. Esto es similar a cómo funcionan los modelos basados en servidor, que transmiten los tokens al usuario, pero sin la latencia de las solicitudes de red.

En cuanto a la interacción con la interfaz, las dos funciones clave son `loadModel()`, que inicializa el LLM, y `fetchAIResponse()`, que procesa la entrada del usuario y genera respuestas de la IA.

### Consideraciones sobre el formato del modelo

> **Important:** Los modelos Phi para MLX no pueden usarse en su formato predeterminado o GGUF. Deben convertirse al formato MLX, proceso que realiza la comunidad MLX. Puedes encontrar modelos preconvertidos en [huggingface.co/mlx-community](https://huggingface.co/mlx-community).

El paquete MLX Examples incluye registros preconfigurados para varios modelos, incluyendo Phi-3. Cuando llamas a `ModelRegistry.phi3_5_4bit`, se referencia un modelo MLX preconvertido específico que se descargará automáticamente:

```swift
static public let phi3_5_4bit = ModelConfiguration(
    id: "mlx-community/Phi-3.5-mini-instruct-4bit",
    defaultPrompt: "What is the gravity on Mars and the moon?",
    extraEOSTokens: ["<|end|>"]
)
```

Puedes crear tus propias configuraciones de modelo para apuntar a cualquier modelo compatible en Hugging Face. Por ejemplo, para usar Phi-4 mini en su lugar, podrías definir tu propia configuración:

```swift
let phi4_mini_4bit = ModelConfiguration(
    id: "mlx-community/Phi-4-mini-instruct-4bit",
    defaultPrompt: "Explain quantum computing in simple terms.",
    extraEOSTokens: ["<|end|>"]
)

// Then use this configuration when loading the model
self.modelContainer = try await LLMModelFactory.shared.loadContainer(
    configuration: phi4_mini_4bit
) { progress in
    print("Download progress: \(Int(progress.fractionCompleted * 100))%")
}
```

> **Note:** El soporte para Phi-4 se añadió al repositorio MLX Swift Examples a finales de febrero de 2025 (en [PR #216](https://github.com/ml-explore/mlx-swift-examples/pull/216)). A marzo de 2025, la última versión oficial (2.21.2 de diciembre de 2024) no incluye soporte incorporado para Phi-4. Para usar modelos Phi-4, deberás referenciar el paquete directamente desde la rama principal:
>
>```swift
> // In your Package.swift or via Xcode's package manager interface
> .package(url: "https://github.com/ml-explore/mlx-swift-examples.git", branch: "main")
> ```

Esto te da acceso a las configuraciones de modelo más recientes, incluyendo Phi-4, antes de que se incluyan en una versión oficial. Puedes usar este método para emplear diferentes versiones de modelos Phi o incluso otros modelos convertidos al formato MLX.

## Paso 6: Crear la interfaz de usuario

Ahora implementemos una interfaz de chat sencilla para interactuar con nuestro view model:

```swift
import SwiftUI

struct ContentView: View {
    @ObservedObject var viewModel = PhiViewModel()

    var body: some View {
        NavigationStack {
            if !viewModel.isReady {
                Spacer()
                if viewModel.isLoadingEngine {
                    ProgressView()
                } else {
                    Button("Load model") {
                        Task {
                            await viewModel.loadModel()
                        }
                    }
                }
                Spacer()
            } else {
                VStack(spacing: 0) {
                    ScrollViewReader { proxy in
                        ScrollView {
                            VStack(alignment: .leading, spacing: 8) {
                                ForEach(viewModel.messages) { message in
                                    MessageView(message: message).padding(.bottom)
                                }
                            }
                            .id("wrapper").padding()
                            .padding()
                        }
                        .onChange(of: viewModel.messages.last?.id, perform: { value in
                            if viewModel.isLoading {
                                proxy.scrollTo("wrapper", anchor: .bottom)
                            } else if let lastMessage = viewModel.messages.last {
                                proxy.scrollTo(lastMessage.id, anchor: .bottom)
                            }
                            
                        })
                    }
                    
                    HStack {
                        TextField("Type a question...", text: $viewModel.prompt, onCommit: {
                            Task {
                                await viewModel.fetchAIResponse()
                            }
                        })
                        .padding(10)
                        .background(Color.gray.opacity(0.2))
                        .cornerRadius(20)
                        .padding(.horizontal)
                        
                        Button(action: {
                            Task {
                                await viewModel.fetchAIResponse()
                            }
                        }) {
                            Image(systemName: "paperplane.fill")
                                .font(.system(size: 24))
                                .foregroundColor(.blue)
                        }
                        .padding(.trailing)
                    }
                    .padding(.bottom)
                }
            }
        }.navigationTitle("Phi Sample")
    }
}

struct MessageView: View {
    let message: ChatMessage

    var body: some View {
        HStack {
            if message.isUser {
                Spacer()
                Text(message.text)
                    .padding()
                    .background(Color.blue)
                    .foregroundColor(.white)
                    .cornerRadius(10)
            } else {
                if message.state == .waiting {
                    TypingIndicatorView()
                } else {
                    VStack {
                        Text(message.text)
                            .padding()
                    }
                    .background(Color.gray.opacity(0.1))
                    .cornerRadius(10)
                    Spacer()
                }
            }
        }
        .padding(.horizontal)
    }
}

struct TypingIndicatorView: View {
    @State private var shouldAnimate = false

    var body: some View {
        HStack {
            ForEach(0..<3) { index in
                Circle()
                    .frame(width: 10, height: 10)
                    .foregroundColor(.gray)
                    .offset(y: shouldAnimate ? -5 : 0)
                    .animation(
                        Animation.easeInOut(duration: 0.5)
                            .repeatForever()
                            .delay(Double(index) * 0.2)
                    )
            }
        }
        .onAppear { shouldAnimate = true }
        .onDisappear { shouldAnimate = false }
    }
}

```

La UI consta de tres componentes principales que trabajan juntos para crear una interfaz básica de chat. `ContentView` crea una interfaz de dos estados que muestra un botón de carga o la interfaz de chat según la disponibilidad del modelo. `MessageView` renderiza los mensajes de chat individualmente, diferenciando si son mensajes del usuario (alineados a la derecha, fondo azul) o respuestas del modelo Phi (alineados a la izquierda, fondo gris). `TypingIndicatorView` ofrece un indicador animado simple para mostrar que la IA está procesando.

## Paso 7: Compilar y ejecutar la app

Ya estamos listos para compilar y ejecutar la aplicación.

> **Important!** MLX no soporta el simulador. Debes ejecutar la app en un dispositivo físico con chip Apple Silicon. Consulta [aquí](https://swiftpackageindex.com/ml-explore/mlx-swift/main/documentation/mlx/running-on-ios#Developing-for-iOS) para más información.

Cuando la app se inicie, toca el botón "Load model" para descargar e inicializar el modelo Phi-3 (o, según tu configuración, Phi-4). Este proceso puede tardar un poco dependiendo de tu conexión a internet, ya que implica descargar el modelo desde Hugging Face. Nuestra implementación solo incluye un spinner para indicar la carga, pero puedes ver el progreso real en la consola de Xcode.

Una vez cargado, puedes interactuar con el modelo escribiendo preguntas en el campo de texto y pulsando el botón de enviar.

Así debería comportarse nuestra aplicación, ejecutándose en un iPad Air M1:

![Demo GIF](../../../../../imgs/01/01/01.phi3ipados.gif)

## Conclusión

¡Y eso es todo! Siguiendo estos pasos, has creado una aplicación iOS que ejecuta el modelo Phi-3 (o Phi-4) directamente en el dispositivo usando el framework MLX de Apple.

¡Felicidades!

**Aviso legal**:  
Este documento ha sido traducido utilizando el servicio de traducción automática [Co-op Translator](https://github.com/Azure/co-op-translator). Aunque nos esforzamos por la precisión, tenga en cuenta que las traducciones automáticas pueden contener errores o inexactitudes. El documento original en su idioma nativo debe considerarse la fuente autorizada. Para información crítica, se recomienda la traducción profesional realizada por humanos. No nos hacemos responsables de malentendidos o interpretaciones erróneas derivadas del uso de esta traducción.