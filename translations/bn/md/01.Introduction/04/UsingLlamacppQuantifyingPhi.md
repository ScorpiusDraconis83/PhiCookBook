<!--
CO_OP_TRANSLATOR_METADATA:
{
  "original_hash": "462bddc47427d8785f3c9fd817b346fe",
  "translation_date": "2025-07-16T22:07:40+00:00",
  "source_file": "md/01.Introduction/04/UsingLlamacppQuantifyingPhi.md",
  "language_code": "bn"
}
-->
# **llama.cpp ব্যবহার করে Phi পরিবার কোয়ান্টাইজেশন**

## **llama.cpp কী**

llama.cpp একটি ওপেন-সোর্স সফটওয়্যার লাইব্রেরি যা প্রধানত C++ এ লেখা হয়েছে এবং বিভিন্ন বড় ভাষা মডেলের (LLMs) উপর ইনফারেন্স করে, যেমন Llama। এর প্রধান উদ্দেশ্য হলো কম সেটআপের মাধ্যমে বিভিন্ন হার্ডওয়্যারে সর্বাধুনিক পারফরম্যান্স প্রদান করা। এছাড়াও, এই লাইব্রেরির জন্য পাইথন বাইন্ডিংস রয়েছে, যা টেক্সট কমপ্লিশনের জন্য উচ্চ-স্তরের API এবং OpenAI সামঞ্জস্যপূর্ণ ওয়েব সার্ভার প্রদান করে।

llama.cpp এর মূল লক্ষ্য হলো কম সেটআপে এবং বিভিন্ন ধরনের হার্ডওয়্যারে—লোকালি এবং ক্লাউডে—LLM ইনফারেন্স সক্ষম করা।

- কোনো ডিপেন্ডেন্সি ছাড়া সরল C/C++ ইমপ্লিমেন্টেশন
- Apple সিলিকনকে প্রথম শ্রেণীর নাগরিক হিসেবে বিবেচনা করা হয়েছে—ARM NEON, Accelerate এবং Metal ফ্রেমওয়ার্কের মাধ্যমে অপ্টিমাইজ করা হয়েছে
- x86 আর্কিটেকচারের জন্য AVX, AVX2 এবং AVX512 সাপোর্ট
- দ্রুত ইনফারেন্স এবং কম মেমরি ব্যবহারের জন্য 1.5-বিট, 2-বিট, 3-বিট, 4-বিট, 5-বিট, 6-বিট এবং 8-বিট ইন্টিজার কোয়ান্টাইজেশন
- NVIDIA GPU-তে LLM চালানোর জন্য কাস্টম CUDA কার্নেল (AMD GPU সাপোর্টের জন্য HIP)
- Vulkan এবং SYCL ব্যাকএন্ড সাপোর্ট
- মোট VRAM ক্ষমতার চেয়ে বড় মডেল আংশিকভাবে দ্রুত করার জন্য CPU+GPU হাইব্রিড ইনফারেন্স

## **llama.cpp দিয়ে Phi-3.5 কোয়ান্টাইজেশন**

Phi-3.5-Instruct মডেলটি llama.cpp ব্যবহার করে কোয়ান্টাইজ করা যায়, কিন্তু Phi-3.5-Vision এবং Phi-3.5-MoE এখনও সাপোর্ট করে না। llama.cpp দ্বারা রূপান্তরিত ফরম্যাট হলো gguf, যা সবচেয়ে বেশি ব্যবহৃত কোয়ান্টাইজেশন ফরম্যাট।

Hugging Face-এ অনেক কোয়ান্টাইজড GGUF ফরম্যাট মডেল পাওয়া যায়। AI Foundry, Ollama, এবং LlamaEdge llama.cpp-র উপর নির্ভরশীল, তাই GGUF মডেলগুলোও প্রায়ই ব্যবহৃত হয়।

### **GGUF কী**

GGUF একটি বাইনারি ফরম্যাট যা মডেল দ্রুত লোড এবং সেভ করার জন্য অপ্টিমাইজ করা হয়েছে, ফলে এটি ইনফারেন্সের জন্য খুবই কার্যকর। GGUF GGML এবং অন্যান্য এক্সিকিউটরের সাথে ব্যবহারের জন্য ডিজাইন করা হয়েছে। GGUF তৈরি করেছেন @ggerganov, যিনি llama.cpp-র ডেভেলপারও। PyTorch-এর মতো ফ্রেমওয়ার্কে প্রথমে তৈরি মডেলগুলো GGUF ফরম্যাটে রূপান্তর করা যায় এই ইঞ্জিনগুলোর সাথে ব্যবহারের জন্য।

### **ONNX বনাম GGUF**

ONNX একটি প্রচলিত মেশিন লার্নিং/ডিপ লার্নিং ফরম্যাট, যা বিভিন্ন AI ফ্রেমওয়ার্কে ভালো সাপোর্ট পায় এবং এজ ডিভাইসেও ভালো ব্যবহারযোগ্য। GGUF হলো llama.cpp-র ভিত্তিতে তৈরি এবং বলা যায় এটি GenAI যুগের ফরম্যাট। উভয়ের ব্যবহার ক্ষেত্র প্রায় একই রকম। এমবেডেড হার্ডওয়্যার এবং অ্যাপ্লিকেশন স্তরে ভালো পারফরম্যান্স চাইলে ONNX হতে পারে আপনার পছন্দ। আর যদি llama.cpp-র ডেরিভেটিভ ফ্রেমওয়ার্ক এবং প্রযুক্তি ব্যবহার করেন, তাহলে GGUF ভালো হতে পারে।

### **llama.cpp দিয়ে Phi-3.5-Instruct কোয়ান্টাইজেশন**

**1. পরিবেশ কনফিগারেশন**


```bash

git clone https://github.com/ggerganov/llama.cpp.git

cd llama.cpp

make -j8

```


**2. কোয়ান্টাইজেশন**

llama.cpp ব্যবহার করে Phi-3.5-Instruct কে FP16 GGUF তে রূপান্তর করুন


```bash

./convert_hf_to_gguf.py <Your Phi-3.5-Instruct Location> --outfile phi-3.5-128k-mini_fp16.gguf

```

Phi-3.5 কে INT4 তে কোয়ান্টাইজ করা


```bash

./llama.cpp/llama-quantize <Your phi-3.5-128k-mini_fp16.gguf location> ./gguf/phi-3.5-128k-mini_Q4_K_M.gguf Q4_K_M

```


**3. পরীক্ষা**

llama-cpp-python ইনস্টল করুন


```bash

pip install llama-cpp-python -U

```

***[!NOTE]***

যদি Apple Silicon ব্যবহার করেন, তাহলে llama-cpp-python এর ইনস্টলেশন এভাবে করুন


```bash

CMAKE_ARGS="-DLLAMA_METAL=on" pip install llama-cpp-python -U

```

পরীক্ষা করুন


```bash

llama.cpp/llama-cli --model <Your phi-3.5-128k-mini_Q4_K_M.gguf location> --prompt "<|user|>\nCan you introduce .NET<|end|>\n<|assistant|>\n"  --gpu-layers 10

```



## **সংশ্লিষ্ট রিসোর্স**

1. llama.cpp সম্পর্কে আরও জানুন [https://github.com/ggml-org/llama.cpp](https://github.com/ggml-org/llama.cpp)
2. onnxruntime সম্পর্কে আরও জানুন [https://onnxruntime.ai/docs/genai/](https://onnxruntime.ai/docs/genai/)
3. GGUF সম্পর্কে আরও জানুন [https://huggingface.co/docs/hub/en/gguf](https://huggingface.co/docs/hub/en/gguf)

**অস্বীকৃতি**:  
এই নথিটি AI অনুবাদ সেবা [Co-op Translator](https://github.com/Azure/co-op-translator) ব্যবহার করে অনূদিত হয়েছে। আমরা যথাসাধ্য সঠিকতার চেষ্টা করি, তবে স্বয়ংক্রিয় অনুবাদে ত্রুটি বা অসঙ্গতি থাকতে পারে। মূল নথিটি তার নিজস্ব ভাষায়ই কর্তৃত্বপূর্ণ উৎস হিসেবে বিবেচিত হওয়া উচিত। গুরুত্বপূর্ণ তথ্যের জন্য পেশাদার মানব অনুবাদ গ্রহণ করার পরামর্শ দেওয়া হয়। এই অনুবাদের ব্যবহারে সৃষ্ট কোনো ভুল বোঝাবুঝি বা ভুল ব্যাখ্যার জন্য আমরা দায়ী নই।