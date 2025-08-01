<!--
CO_OP_TRANSLATOR_METADATA:
{
  "original_hash": "462bddc47427d8785f3c9fd817b346fe",
  "translation_date": "2025-07-16T22:11:14+00:00",
  "source_file": "md/01.Introduction/04/UsingLlamacppQuantifyingPhi.md",
  "language_code": "tl"
}
-->
# **Pag-quantize ng Phi Family gamit ang llama.cpp**

## **Ano ang llama.cpp**

Ang llama.cpp ay isang open-source na software library na pangunahing nakasulat sa C++ na nagsasagawa ng inference sa iba't ibang Large Language Models (LLMs), tulad ng Llama. Ang pangunahing layunin nito ay magbigay ng makabagong performance para sa LLM inference sa iba't ibang hardware na may minimal na setup. Bukod dito, may mga Python bindings din para sa library na ito, na nag-aalok ng high-level API para sa text completion at isang OpenAI compatible na web server.

Ang pangunahing layunin ng llama.cpp ay payagan ang LLM inference na may minimal na setup at makabagong performance sa iba't ibang hardware - lokal man o sa cloud.

- Plain C/C++ implementation na walang anumang dependencies
- Apple silicon ay isang first-class citizen - optimized gamit ang ARM NEON, Accelerate, at Metal frameworks
- Suporta para sa AVX, AVX2, at AVX512 para sa x86 architectures
- 1.5-bit, 2-bit, 3-bit, 4-bit, 5-bit, 6-bit, at 8-bit integer quantization para sa mas mabilis na inference at mas mababang paggamit ng memorya
- Custom CUDA kernels para sa pagpapatakbo ng LLMs sa NVIDIA GPUs (suporta para sa AMD GPUs gamit ang HIP)
- Suporta sa Vulkan at SYCL backend
- CPU+GPU hybrid inference para bahagyang pabilisin ang mga modelong mas malaki kaysa sa kabuuang VRAM capacity

## **Pag-quantize ng Phi-3.5 gamit ang llama.cpp**

Maaaring i-quantize ang Phi-3.5-Instruct model gamit ang llama.cpp, ngunit ang Phi-3.5-Vision at Phi-3.5-MoE ay hindi pa suportado. Ang format na kinokonvert ng llama.cpp ay gguf, na siyang pinakaginagamit na quantization format.

Maraming mga quantized GGUF format models sa Hugging Face. Ang AI Foundry, Ollama, at LlamaEdge ay umaasa sa llama.cpp, kaya madalas ding ginagamit ang mga GGUF models.

### **Ano ang GGUF**

Ang GGUF ay isang binary format na optimized para sa mabilis na pag-load at pag-save ng mga modelo, kaya napaka-epektibo para sa inference. Ang GGUF ay dinisenyo para gamitin kasama ang GGML at iba pang executors. Ang GGUF ay binuo ni @ggerganov na siya ring developer ng llama.cpp, isang kilalang C/C++ LLM inference framework. Ang mga modelong unang ginawa sa mga framework tulad ng PyTorch ay maaaring i-convert sa GGUF format para magamit sa mga engine na iyon.

### **ONNX vs GGUF**

Ang ONNX ay isang tradisyunal na machine learning/deep learning format, na mahusay ang suporta sa iba't ibang AI Frameworks at may magagandang gamit sa mga edge devices. Samantalang ang GGUF ay nakabase sa llama.cpp at masasabi na produkto ng GenAI era. Magkapareho ang gamit ng dalawa. Kung nais mo ng mas mahusay na performance sa embedded hardware at application layers, maaaring ONNX ang piliin mo. Kung gagamit ka ng derivative framework at teknolohiya ng llama.cpp, mas mainam ang GGUF.

### **Pag-quantize ng Phi-3.5-Instruct gamit ang llama.cpp**

**1. Pag-configure ng Kapaligiran**


```bash

git clone https://github.com/ggerganov/llama.cpp.git

cd llama.cpp

make -j8

```


**2. Pag-quantize**

Gamit ang llama.cpp, i-convert ang Phi-3.5-Instruct sa FP16 GGUF


```bash

./convert_hf_to_gguf.py <Your Phi-3.5-Instruct Location> --outfile phi-3.5-128k-mini_fp16.gguf

```

Pag-quantize ng Phi-3.5 sa INT4


```bash

./llama.cpp/llama-quantize <Your phi-3.5-128k-mini_fp16.gguf location> ./gguf/phi-3.5-128k-mini_Q4_K_M.gguf Q4_K_M

```


**3. Pagsubok**

I-install ang llama-cpp-python


```bash

pip install llama-cpp-python -U

```

***Tandaan*** 

Kung gumagamit ka ng Apple Silicon, i-install ang llama-cpp-python sa ganitong paraan


```bash

CMAKE_ARGS="-DLLAMA_METAL=on" pip install llama-cpp-python -U

```

Pagsubok 


```bash

llama.cpp/llama-cli --model <Your phi-3.5-128k-mini_Q4_K_M.gguf location> --prompt "<|user|>\nCan you introduce .NET<|end|>\n<|assistant|>\n"  --gpu-layers 10

```



## **Mga Sanggunian**

1. Alamin pa ang tungkol sa llama.cpp [https://github.com/ggml-org/llama.cpp](https://github.com/ggml-org/llama.cpp)
2. Alamin pa ang tungkol sa onnxruntime [https://onnxruntime.ai/docs/genai/](https://onnxruntime.ai/docs/genai/)
3. Alamin pa ang tungkol sa GGUF [https://huggingface.co/docs/hub/en/gguf](https://huggingface.co/docs/hub/en/gguf)

**Paalala**:  
Ang dokumentong ito ay isinalin gamit ang AI translation service na [Co-op Translator](https://github.com/Azure/co-op-translator). Bagamat nagsusumikap kami para sa katumpakan, pakatandaan na ang mga awtomatikong pagsasalin ay maaaring maglaman ng mga pagkakamali o di-tumpak na impormasyon. Ang orihinal na dokumento sa orihinal nitong wika ang dapat ituring na pangunahing sanggunian. Para sa mahahalagang impormasyon, inirerekomenda ang propesyonal na pagsasalin ng tao. Hindi kami mananagot sa anumang hindi pagkakaunawaan o maling interpretasyon na maaaring magmula sa paggamit ng pagsasaling ito.