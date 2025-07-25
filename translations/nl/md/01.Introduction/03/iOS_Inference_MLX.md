<!--
CO_OP_TRANSLATOR_METADATA:
{
  "original_hash": "9a626d7522772d8b7b6f188dc79108c4",
  "translation_date": "2025-07-16T20:33:46+00:00",
  "source_file": "md/01.Introduction/03/iOS_Inference_MLX.md",
  "language_code": "nl"
}
-->
# Phi-3 en Phi-4 draaien op iOS met het Apple MLX Framework

Deze tutorial laat zien hoe je een iOS-app maakt die het Phi-3 of Phi-4 model lokaal op het apparaat uitvoert, met behulp van het Apple MLX framework. [MLX](https://opensource.apple.com/projects/mlx/) is Apple’s machine learning framework, geoptimaliseerd voor Apple Silicon chips.

## Vereisten

- macOS met Xcode 16 (of hoger)  
- iOS 18 (of hoger) als doelapparaat met minimaal 8GB (iPhone of iPad compatibel met Apple Intelligence vereisten, vergelijkbaar met de gequantiseerde Phi vereisten)  
- basiskennis van Swift en SwiftUI  

## Stap 1: Maak een nieuw iOS-project aan

Begin met het aanmaken van een nieuw iOS-project in Xcode:

1. start Xcode en kies "Create a new Xcode project"  
2. selecteer "App" als template  
3. geef je project een naam (bijv. "Phi3-iOS-App") en kies SwiftUI als interface  
4. kies een locatie om je project op te slaan  

## Stap 2: Voeg de benodigde dependencies toe

Voeg het [MLX Examples package](https://github.com/ml-explore/mlx-swift-examples) toe, dat alle benodigde dependencies en helpers bevat voor het vooraf laden van modellen en het uitvoeren van inferentie:

```swift
// In Xcode: File > Add Package Dependencies
// URL: https://github.com/ml-explore/mlx-swift-examples
```

Hoewel het basis [MLX Swift package](https://github.com/ml-explore/mlx-swift) voldoende is voor kern-tensorbewerkingen en basis ML-functionaliteit, biedt het MLX Examples package extra componenten die speciaal zijn ontworpen voor het werken met taalmodellen en het vereenvoudigen van het inferentieproces:

- utilities voor het laden van modellen die downloaden vanaf Hugging Face afhandelen  
- integratie van tokenizers  
- helpers voor tekstgeneratie tijdens inferentie  
- vooraf geconfigureerde modeldefinities  

## Stap 3: Configureer entitlements

Om onze app toe te staan modellen te downloaden en voldoende geheugen toe te wijzen, moeten we specifieke entitlements toevoegen. Maak een `.entitlements` bestand aan voor je app met de volgende inhoud:

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

> **Note:** De `com.apple.developer.kernel.increased-memory-limit` entitlement is belangrijk voor het draaien van grotere modellen, omdat het de app toestaat meer geheugen aan te vragen dan normaal is toegestaan.

## Stap 4: Maak het Chat Message model

Laten we eerst een eenvoudige structuur maken om onze chatberichten te representeren:

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

## Stap 5: Implementeer de ViewModel

Vervolgens maken we de `PhiViewModel` klasse die het laden van het model en de inferentie afhandelt:

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

De ViewModel laat de belangrijkste MLX-integratiepunten zien:

- het instellen van GPU cache limieten met `MLX.GPU.set(cacheLimit:)` om het geheugengebruik op mobiele apparaten te optimaliseren  
- gebruik van `LLMModelFactory` om het model on-demand te downloaden en het MLX-geoptimaliseerde model te initialiseren  
- toegang tot de parameters en structuur van het model via de `ModelContainer`  
- gebruik van MLX’s token-voor-token generatie via de `MLXLMCommon.generate` methode  
- het beheren van het inferentieproces met passende temperatuurinstellingen en tokenlimieten  

De streaming token generatie geeft gebruikers directe feedback terwijl het model tekst genereert. Dit lijkt op hoe servergebaseerde modellen werken, die tokens terug streamen naar de gebruiker, maar dan zonder de vertraging van netwerkverzoeken.

Qua UI-interactie zijn de twee belangrijkste functies `loadModel()`, die de LLM initialiseert, en `fetchAIResponse()`, die gebruikersinvoer verwerkt en AI-antwoorden genereert.

### Overwegingen voor het modelformaat

> **Important:** Phi-modellen voor MLX kunnen niet in hun standaard- of GGUF-formaat worden gebruikt. Ze moeten worden geconverteerd naar het MLX-formaat, wat wordt verzorgd door de MLX-community. Je kunt vooraf geconverteerde modellen vinden op [huggingface.co/mlx-community](https://huggingface.co/mlx-community).

Het MLX Examples package bevat vooraf geconfigureerde registraties voor verschillende modellen, waaronder Phi-3. Wanneer je `ModelRegistry.phi3_5_4bit` aanroept, verwijst dit naar een specifiek vooraf geconverteerd MLX-model dat automatisch wordt gedownload:

```swift
static public let phi3_5_4bit = ModelConfiguration(
    id: "mlx-community/Phi-3.5-mini-instruct-4bit",
    defaultPrompt: "What is the gravity on Mars and the moon?",
    extraEOSTokens: ["<|end|>"]
)
```

Je kunt ook je eigen modelconfiguraties maken die verwijzen naar elk compatibel model op Hugging Face. Bijvoorbeeld, om Phi-4 mini te gebruiken, kun je je eigen configuratie definiëren:

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

> **Note:** Phi-4 ondersteuning is toegevoegd aan de MLX Swift Examples repository eind februari 2025 (in [PR #216](https://github.com/ml-explore/mlx-swift-examples/pull/216)). Vanaf maart 2025 bevat de laatste officiële release (2.21.2 van december 2024) nog geen ingebouwde Phi-4 ondersteuning. Om Phi-4 modellen te gebruiken, moet je het package rechtstreeks vanuit de main branch refereren:  
>  
>```swift
> // In your Package.swift or via Xcode's package manager interface
> .package(url: "https://github.com/ml-explore/mlx-swift-examples.git", branch: "main")
> ```

Dit geeft je toegang tot de nieuwste modelconfiguraties, inclusief Phi-4, voordat ze in een officiële release worden opgenomen. Je kunt deze aanpak gebruiken om verschillende versies van Phi-modellen te gebruiken of zelfs andere modellen die naar het MLX-formaat zijn geconverteerd.

## Stap 6: Maak de UI

Laten we nu een eenvoudige chatinterface implementeren om met onze view model te communiceren:

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

De UI bestaat uit drie hoofdcomponenten die samenwerken om een basis chatinterface te creëren. `ContentView` maakt een interface met twee toestanden die ofwel een laadknop toont of de chatinterface, afhankelijk van de gereedheid van het model. `MessageView` toont individuele chatberichten verschillend, afhankelijk van of het gebruikersberichten zijn (rechts uitgelijnd, blauwe achtergrond) of Phi modelantwoorden (links uitgelijnd, grijze achtergrond). `TypingIndicatorView` biedt een eenvoudige geanimeerde indicator om te laten zien dat de AI aan het verwerken is.

## Stap 7: Bouw en start de app

We zijn nu klaar om de applicatie te bouwen en te starten.

> **Important!** MLX ondersteunt de simulator niet. Je moet de app draaien op een fysiek apparaat met een Apple Silicon chip. Zie [hier](https://swiftpackageindex.com/ml-explore/mlx-swift/main/documentation/mlx/running-on-ios#Developing-for-iOS) voor meer informatie.

Wanneer de app start, tik je op de knop "Load model" om het Phi-3 (of, afhankelijk van je configuratie, Phi-4) model te downloaden en te initialiseren. Dit kan even duren, afhankelijk van je internetverbinding, omdat het model van Hugging Face wordt gedownload. Onze implementatie toont alleen een spinner tijdens het laden, maar je kunt de voortgang zien in de Xcode-console.

Zodra het geladen is, kun je met het model communiceren door vragen te typen in het tekstveld en op de verzendknop te drukken.

Zo zou onze applicatie moeten werken, hier draaiend op een iPad Air M1:

![Demo GIF](../../../../../imgs/01/01/01.phi3ipados.gif)

## Conclusie

En dat is het! Door deze stappen te volgen, heb je een iOS-app gemaakt die het Phi-3 (of Phi-4) model direct op het apparaat uitvoert met behulp van Apple’s MLX framework.

Gefeliciteerd!

**Disclaimer**:  
Dit document is vertaald met behulp van de AI-vertalingsdienst [Co-op Translator](https://github.com/Azure/co-op-translator). Hoewel we streven naar nauwkeurigheid, dient u er rekening mee te houden dat geautomatiseerde vertalingen fouten of onnauwkeurigheden kunnen bevatten. Het originele document in de oorspronkelijke taal moet als de gezaghebbende bron worden beschouwd. Voor cruciale informatie wordt professionele menselijke vertaling aanbevolen. Wij zijn niet aansprakelijk voor eventuele misverstanden of verkeerde interpretaties die voortvloeien uit het gebruik van deze vertaling.