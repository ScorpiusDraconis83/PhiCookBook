<!--
CO_OP_TRANSLATOR_METADATA:
{
  "original_hash": "2b94610e2f6fe648e01fa23626f0dd03",
  "translation_date": "2025-07-17T07:56:43+00:00",
  "source_file": "md/03.FineTuning/FineTuning_MLX.md",
  "language_code": "ru"
}
-->
# **Тонкая настройка Phi-3 с помощью Apple MLX Framework**

Мы можем выполнить тонкую настройку в сочетании с Lora через командную строку Apple MLX Framework. (Если вы хотите узнать больше о работе MLX Framework, пожалуйста, прочитайте [Inference Phi-3 with Apple MLX Framework](../03.FineTuning/03.Inference/MLX_Inference.md))


## **1. Подготовка данных**

По умолчанию MLX Framework требует формат jsonl для train, test и eval, и используется вместе с Lora для выполнения задач тонкой настройки.


### ***Примечание:***

1. Формат данных jsonl:


```json

{"text": "<|user|>\nWhen were iron maidens commonly used? <|end|>\n<|assistant|> \nIron maidens were never commonly used <|end|>"}
{"text": "<|user|>\nWhat did humans evolve from? <|end|>\n<|assistant|> \nHumans and apes evolved from a common ancestor <|end|>"}
{"text": "<|user|>\nIs 91 a prime number? <|end|>\n<|assistant|> \nNo, 91 is not a prime number <|end|>"}
....

```

2. В нашем примере используются данные [TruthfulQA](https://github.com/sylinrl/TruthfulQA/blob/main/TruthfulQA.csv), но объём данных относительно небольшой, поэтому результаты тонкой настройки могут быть не оптимальными. Рекомендуется использовать более качественные данные, исходя из ваших задач.

3. Формат данных соответствует шаблону Phi-3

Пожалуйста, скачайте данные по этой [ссылке](../../../../code/04.Finetuning/mlx), включите все файлы .jsonl из папки ***data***


## **2. Тонкая настройка в терминале**

Запустите эту команду в терминале


```bash

python -m mlx_lm.lora --model microsoft/Phi-3-mini-4k-instruct --train --data ./data --iters 1000 

```


## ***Примечание:***

1. Это тонкая настройка LoRA, MLX Framework не поддерживает QLoRA

2. Вы можете изменить некоторые параметры в config.yaml, например


```yaml


# The path to the local model directory or Hugging Face repo.
model: "microsoft/Phi-3-mini-4k-instruct"
# Whether or not to train (boolean)
train: true

# Directory with {train, valid, test}.jsonl files
data: "data"

# The PRNG seed
seed: 0

# Number of layers to fine-tune
lora_layers: 32

# Minibatch size.
batch_size: 1

# Iterations to train for.
iters: 1000

# Number of validation batches, -1 uses the entire validation set.
val_batches: 25

# Adam learning rate.
learning_rate: 1e-6

# Number of training steps between loss reporting.
steps_per_report: 10

# Number of training steps between validations.
steps_per_eval: 200

# Load path to resume training with the given adapter weights.
resume_adapter_file: null

# Save/load path for the trained adapter weights.
adapter_path: "adapters"

# Save the model every N iterations.
save_every: 1000

# Evaluate on the test set after training
test: false

# Number of test set batches, -1 uses the entire test set.
test_batches: 100

# Maximum sequence length.
max_seq_length: 2048

# Use gradient checkpointing to reduce memory use.
grad_checkpoint: true

# LoRA parameters can only be specified in a config file
lora_parameters:
  # The layer keys to apply LoRA to.
  # These will be applied for the last lora_layers
  keys: ["o_proj","qkv_proj"]
  rank: 64
  scale: 1
  dropout: 0.1


```

Запустите эту команду в терминале


```bash

python -m  mlx_lm.lora --config lora_config.yaml

```


## **3. Запуск адаптера тонкой настройки для тестирования**

Вы можете запустить адаптер тонкой настройки в терминале, например так


```bash

python -m mlx_lm.generate --model microsoft/Phi-3-mini-4k-instruct --adapter-path ./adapters --max-token 2048 --prompt "Why do chameleons change colors? " --eos-token "<|end|>"    

```

и запустить оригинальную модель для сравнения результатов


```bash

python -m mlx_lm.generate --model microsoft/Phi-3-mini-4k-instruct --max-token 2048 --prompt "Why do chameleons change colors? " --eos-token "<|end|>"    

```

Попробуйте сравнить результаты тонкой настройки с оригинальной моделью


## **4. Объединение адаптеров для создания новых моделей**


```bash

python -m mlx_lm.fuse --model microsoft/Phi-3-mini-4k-instruct

```

## **5. Запуск квантованных моделей тонкой настройки с помощью ollama**

Перед использованием настройте окружение llama.cpp


```bash

git clone https://github.com/ggerganov/llama.cpp.git

cd llama.cpp

pip install -r requirements.txt

python convert.py 'Your meger model path'  --outfile phi-3-mini-ft.gguf --outtype f16 

```

***Примечание:***

1. Сейчас поддерживается квантование из fp32, fp16 и INT 8

2. В объединённой модели отсутствует tokenizer.model, скачайте его с https://huggingface.co/microsoft/Phi-3-mini-4k-instruct

установите [Ollma Model](https://ollama.com/)


```txt

FROM ./phi-3-mini-ft.gguf
PARAMETER stop "<|end|>"

```

Запустите команду в терминале


```bash

 ollama create phi3ft -f Modelfile 

 ollama run phi3ft "Why do chameleons change colors?" 

```

Поздравляем! Вы освоили тонкую настройку с MLX Framework

**Отказ от ответственности**:  
Этот документ был переведен с помощью сервиса автоматического перевода [Co-op Translator](https://github.com/Azure/co-op-translator). Несмотря на наши усилия по обеспечению точности, просим учитывать, что автоматический перевод может содержать ошибки или неточности. Оригинальный документ на его исходном языке следует считать авторитетным источником. Для получения критически важной информации рекомендуется обращаться к профессиональному человеческому переводу. Мы не несем ответственности за любые недоразумения или неправильные толкования, возникшие в результате использования данного перевода.