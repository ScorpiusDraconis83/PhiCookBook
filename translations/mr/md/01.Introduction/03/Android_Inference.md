<!--
CO_OP_TRANSLATOR_METADATA:
{
  "original_hash": "9481b07dda8f9715a5d1ff43fb27568b",
  "translation_date": "2025-07-16T20:12:33+00:00",
  "source_file": "md/01.Introduction/03/Android_Inference.md",
  "language_code": "mr"
}
-->
# **Android मध्ये Inference Phi-3**

चला पाहूया की Android डिव्हाइसेसवर Phi-3-mini सह inference कसे करता येते. Phi-3-mini ही Microsoft कडून आलेली नवीन मॉडेल्सची मालिका आहे जी Edge डिव्हाइसेस आणि IoT डिव्हाइसेसवर Large Language Models (LLMs) चा वापर शक्य करते.

## Semantic Kernel आणि Inference

[Semantic Kernel](https://github.com/microsoft/semantic-kernel) हा एक application framework आहे जो तुम्हाला Azure OpenAI Service, OpenAI मॉडेल्स आणि अगदी लोकल मॉडेल्ससह सुसंगत अ‍ॅप्लिकेशन्स तयार करण्याची परवानगी देतो. जर तुम्ही Semantic Kernel मध्ये नवीन असाल, तर आम्ही तुम्हाला [Semantic Kernel Cookbook](https://github.com/microsoft/SemanticKernelCookBook?WT.mc_id=aiml-138114-kinfeylo) पाहण्याचा सल्ला देतो.

### Semantic Kernel वापरून Phi-3-mini कसे Access करावे

तुम्ही ते Semantic Kernel मधील Hugging Face Connector सोबत एकत्र करू शकता. यासाठी हा [Sample Code](https://github.com/Azure-Samples/Phi-3MiniSamples/tree/main/semantickernel?WT.mc_id=aiml-138114-kinfeylo) पहा.

डिफॉल्टनुसार, हे Hugging Face वरील मॉडेल ID शी जुळते. पण तुम्ही स्थानिकरित्या तयार केलेल्या Phi-3-mini मॉडेल सर्व्हरशी देखील कनेक्ट करू शकता.

### Ollama किंवा LlamaEdge सह Quantized Models कॉल करणे

अनेक वापरकर्ते मॉडेल्स स्थानिकरित्या चालवण्यासाठी quantized मॉडेल्स वापरणे पसंत करतात. [Ollama](https://ollama.com/) आणि [LlamaEdge](https://llamaedge.com) वैयक्तिक वापरकर्त्यांना वेगवेगळ्या quantized मॉडेल्स कॉल करण्याची परवानगी देतात:

#### Ollama

तुम्ही थेट `ollama run Phi-3` चालवू शकता किंवा `.gguf` फाइलच्या पथासह `Modelfile` तयार करून ऑफलाइन कॉन्फिगर करू शकता.

```gguf
FROM {Add your gguf file path}
TEMPLATE \"\"\"<|user|> .Prompt<|end|> <|assistant|>\"\"\"
PARAMETER stop <|end|>
PARAMETER num_ctx 4096
```

[Sample Code](https://github.com/Azure-Samples/Phi-3MiniSamples/tree/main/ollama?WT.mc_id=aiml-138114-kinfeylo)

#### LlamaEdge

जर तुम्हाला क्लाउड आणि Edge डिव्हाइसेसवर एकाच वेळी `.gguf` फाइल्स वापरायच्या असतील, तर LlamaEdge हा उत्तम पर्याय आहे. सुरुवात करण्यासाठी हा [sample code](https://github.com/Azure-Samples/Phi-3MiniSamples/tree/main/wasm?WT.mc_id=aiml-138114-kinfeylo) पाहू शकता.

### Android फोनवर इंस्टॉल आणि चालवा

1. **MLC Chat अ‍ॅप डाउनलोड करा** (मुफ्त) Android फोनसाठी.
2. APK फाइल (148MB) डाउनलोड करून तुमच्या डिव्हाइसवर इंस्टॉल करा.
3. MLC Chat अ‍ॅप सुरू करा. तुम्हाला Phi-3-mini सह AI मॉडेल्सची यादी दिसेल.

सारांश म्हणून, Phi-3-mini Edge डिव्हाइसेसवर जनरेटिव्ह AI साठी नवीन शक्यता उघडते, आणि तुम्ही Android वर त्याच्या क्षमतांचा अनुभव घेऊ शकता.

**अस्वीकरण**:  
हा दस्तऐवज AI अनुवाद सेवा [Co-op Translator](https://github.com/Azure/co-op-translator) वापरून अनुवादित केला आहे. आम्ही अचूकतेसाठी प्रयत्नशील असलो तरी, कृपया लक्षात घ्या की स्वयंचलित अनुवादांमध्ये चुका किंवा अचूकतेची कमतरता असू शकते. मूळ दस्तऐवज त्याच्या स्थानिक भाषेत अधिकृत स्रोत मानला जावा. महत्त्वाच्या माहितीसाठी व्यावसायिक मानवी अनुवाद करण्याची शिफारस केली जाते. या अनुवादाच्या वापरामुळे उद्भवलेल्या कोणत्याही गैरसमजुती किंवा चुकीच्या अर्थलागी आम्ही जबाबदार नाही.