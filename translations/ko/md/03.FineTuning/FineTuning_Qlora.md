<!--
CO_OP_TRANSLATOR_METADATA:
{
  "original_hash": "54b6b824568d4decb574b9e117c4f5f7",
  "translation_date": "2025-07-17T08:17:37+00:00",
  "source_file": "md/03.FineTuning/FineTuning_Qlora.md",
  "language_code": "ko"
}
-->
**QLoRA를 이용한 Phi-3 미세 조정**

Microsoft의 Phi-3 Mini 언어 모델을 [QLoRA (Quantum Low-Rank Adaptation)](https://github.com/artidoro/qlora)를 사용해 미세 조정합니다.

QLoRA는 대화 이해와 응답 생성 능력을 향상시키는 데 도움을 줍니다.

transformers와 bitsandbytes를 사용해 4비트 모델을 로드하려면 accelerate와 transformers를 소스에서 설치하고 최신 버전의 bitsandbytes 라이브러리가 설치되어 있는지 확인해야 합니다.

**샘플**
- [이 샘플 노트북으로 더 알아보기](../../../../code/03.Finetuning/Phi_3_Inference_Finetuning.ipynb)
- [Python 미세 조정 샘플 예제](../../../../code/03.Finetuning/FineTrainingScript.py)
- [LORA를 이용한 Hugging Face Hub 미세 조정 예제](../../../../code/03.Finetuning/Phi-3-finetune-lora-python.ipynb)
- [QLORA를 이용한 Hugging Face Hub 미세 조정 예제](../../../../code/03.Finetuning/Phi-3-finetune-qlora-python.ipynb)

**면책 조항**:  
이 문서는 AI 번역 서비스 [Co-op Translator](https://github.com/Azure/co-op-translator)를 사용하여 번역되었습니다. 정확성을 위해 노력하고 있으나, 자동 번역에는 오류나 부정확한 부분이 있을 수 있음을 유의하시기 바랍니다. 원문은 해당 언어의 원본 문서가 권위 있는 출처로 간주되어야 합니다. 중요한 정보의 경우 전문적인 인간 번역을 권장합니다. 본 번역 사용으로 인해 발생하는 오해나 잘못된 해석에 대해 당사는 책임을 지지 않습니다.