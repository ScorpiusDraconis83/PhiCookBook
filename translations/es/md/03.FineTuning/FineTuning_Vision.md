<!--
CO_OP_TRANSLATOR_METADATA:
{
  "original_hash": "a5a67308d3b2c5af97baf01067c6f007",
  "translation_date": "2025-07-17T08:32:50+00:00",
  "source_file": "md/03.FineTuning/FineTuning_Vision.md",
  "language_code": "es"
}
-->
# Receta de ajuste fino de Phi-3.5-vision

Este es el soporte oficial para el ajuste fino de Phi-3.5-vision usando las librerías de huggingface.  
Por favor, `cd` al directorio de código [vision_finetuning](../../../../code/03.Finetuning/vision_finetuning) antes de ejecutar los siguientes comandos.

## Instalación

```bash
# create a new conda environment
conda create -n phi3v python=3.10
conda activate phi3v

# install pytorch
conda install pytorch==2.1.2 torchvision==0.16.2 torchaudio==2.1.2 pytorch-cuda=12.1 -c pytorch -c nvidia

# other libraries needed to run the example code
pip install -r requirements.txt

# (optional) flash attention -- Ampere+ GPUs (e.g., A100, H100)
pip install ninja
MAX_JOBS=32 pip install flash-attn==2.4.2 --no-build-isolation

# (optional) QLoRA -- Turing+ GPUs (e.g., RTX 8000)
pip install bitsandbytes==0.43.1
```

## Inicio rápido

Proporcionamos dos scripts de ejemplo para ajuste fino, uno para DocVQA y otro para clasificación de memes de odio.

Hardware mínimo probado en 4x RTX8000 (48GB de RAM por GPU)

```bash
# minimal script on a mini-train split of DocVQA
torchrun --nproc_per_node=4 finetune_hf_trainer_docvqa.py
```

Phi-3.5-vision ahora soporta oficialmente entradas con múltiples imágenes. Aquí un ejemplo para ajustar finamente NLVR2

```bash
torchrun --nproc_per_node=8 finetune_hf_trainer_nlvr2.py
```

## Guía de uso

Dependiendo del hardware, los usuarios pueden elegir diferentes estrategias de ajuste fino. Soportamos  
ajuste fino completo (con Deepspeed Zero-2) con parámetros de visión opcionalmente congelados, y LoRA (incluyendo QLoRA de 4 bits).  
En general, recomendamos usar ajuste fino completo con flash attention y bf16 siempre que sea posible.

### Guía para convertir tu conjunto de datos personalizado al formato requerido

Usamos un conjunto de datos mínimo de clasificación de video (un subconjunto de UCF-101) como ejemplo de extremo a extremo para demostrar cómo convertir tu conjunto de datos personalizado al formato requerido y ajustar finamente Phi-3.5-vision sobre él.

```bash
# convert data
python convert_ucf101.py --out_dir /path/to/converted_ucf101

# training
torchrun --nproc_per_node=4 finetune_hf_trainer_ucf101.py --data_dir /path/to/converted_ucf101
```

Los datos convertidos se verán así:

```bash
> tree --filelimit=10 /path/to/converted_ucf101
/path/to/converted_ucf101
├── images
│   ├── test
│   │   ├── ApplyEyeMakeup [48 entries exceeds filelimit, not opening dir]
│   │   ├── ApplyLipstick [32 entries exceeds filelimit, not opening dir]
│   │   ├── Archery [56 entries exceeds filelimit, not opening dir]
│   │   ├── BabyCrawling [72 entries exceeds filelimit, not opening dir]
│   │   ├── BalanceBeam [32 entries exceeds filelimit, not opening dir]
│   │   ├── BandMarching [72 entries exceeds filelimit, not opening dir]
│   │   ├── BaseballPitch [80 entries exceeds filelimit, not opening dir]
│   │   ├── Basketball [88 entries exceeds filelimit, not opening dir]
│   │   ├── BasketballDunk [48 entries exceeds filelimit, not opening dir]
│   │   └── BenchPress [72 entries exceeds filelimit, not opening dir]
│   ├── train
│   │   ├── ApplyEyeMakeup [240 entries exceeds filelimit, not opening dir]
│   │   ├── ApplyLipstick [240 entries exceeds filelimit, not opening dir]
│   │   ├── Archery [240 entries exceeds filelimit, not opening dir]
│   │   ├── BabyCrawling [240 entries exceeds filelimit, not opening dir]
│   │   ├── BalanceBeam [240 entries exceeds filelimit, not opening dir]
│   │   ├── BandMarching [240 entries exceeds filelimit, not opening dir]
│   │   ├── BaseballPitch [240 entries exceeds filelimit, not opening dir]
│   │   ├── Basketball [240 entries exceeds filelimit, not opening dir]
│   │   ├── BasketballDunk [240 entries exceeds filelimit, not opening dir]
│   │   └── BenchPress [240 entries exceeds filelimit, not opening dir]
│   └── val
│       ├── ApplyEyeMakeup [24 entries exceeds filelimit, not opening dir]
│       ├── ApplyLipstick [24 entries exceeds filelimit, not opening dir]
│       ├── Archery [24 entries exceeds filelimit, not opening dir]
│       ├── BabyCrawling [24 entries exceeds filelimit, not opening dir]
│       ├── BalanceBeam [24 entries exceeds filelimit, not opening dir]
│       ├── BandMarching [24 entries exceeds filelimit, not opening dir]
│       ├── BaseballPitch [24 entries exceeds filelimit, not opening dir]
│       ├── Basketball [24 entries exceeds filelimit, not opening dir]
│       ├── BasketballDunk [24 entries exceeds filelimit, not opening dir]
│       └── BenchPress [24 entries exceeds filelimit, not opening dir]
├── ucf101_test.jsonl
├── ucf101_train.jsonl
└── ucf101_val.jsonl

34 directories, 3 files
```

Para la anotación `jsonl`, cada línea debe ser un diccionario como:

```json
{"id": "val-0000000300", "source": "ucf101", "conversations": [{"images": ["val/BabyCrawling/v_BabyCrawling_g21_c04.0.jpg", "val/BabyCrawling/v_BabyCrawling_g21_c04.1.jpg", "val/BabyCrawling/v_BabyCrawling_g21_c04.2.jpg", "val/BabyCrawling/v_BabyCrawling_g21_c04.3.jpg", "val/BabyCrawling/v_BabyCrawling_g21_c04.4.jpg", "val/BabyCrawling/v_BabyCrawling_g21_c04.5.jpg", "val/BabyCrawling/v_BabyCrawling_g21_c04.6.jpg", "val/BabyCrawling/v_BabyCrawling_g21_c04.7.jpg"], "user": "Classify the video into one of the following classes: ApplyEyeMakeup, ApplyLipstick, Archery, BabyCrawling, BalanceBeam, BandMarching, BaseballPitch, Basketball, BasketballDunk, BenchPress.", "assistant": "BabyCrawling"}]}
{"id": "val-0000000301", "source": "ucf101", "conversations": [{"images": ["val/BabyCrawling/v_BabyCrawling_g09_c06.0.jpg", "val/BabyCrawling/v_BabyCrawling_g09_c06.1.jpg", "val/BabyCrawling/v_BabyCrawling_g09_c06.2.jpg", "val/BabyCrawling/v_BabyCrawling_g09_c06.3.jpg", "val/BabyCrawling/v_BabyCrawling_g09_c06.4.jpg", "val/BabyCrawling/v_BabyCrawling_g09_c06.5.jpg", "val/BabyCrawling/v_BabyCrawling_g09_c06.6.jpg", "val/BabyCrawling/v_BabyCrawling_g09_c06.7.jpg"], "user": "Classify the video into one of the following classes: ApplyEyeMakeup, ApplyLipstick, Archery, BabyCrawling, BalanceBeam, BandMarching, BaseballPitch, Basketball, BasketballDunk, BenchPress.", "assistant": "BabyCrawling"}]}
```

Ten en cuenta que `conversations` es una lista, por lo que se puede soportar conversación de múltiples turnos si se dispone de esos datos.

## Solicitud de cuota de GPU en Azure

### Requisitos previos

Una cuenta de Azure con el rol de Contributor (o cualquier otro rol que incluya acceso de Contributor).

Si no tienes una cuenta de Azure, crea una [cuenta gratuita antes de comenzar](https://azure.microsoft.com).

### Solicitar un aumento de cuota

Puedes enviar una solicitud para aumentar la cuota directamente desde Mis cuotas. Sigue los pasos a continuación para solicitar un aumento de cuota. Para este ejemplo, puedes seleccionar cualquier cuota ajustable en tu suscripción.

Inicia sesión en el [portal de Azure](https://portal.azure.com).

Escribe "quotas" en el cuadro de búsqueda y luego selecciona Quotas.  
![Quota](https://learn.microsoft.com/azure/quotas/media/quickstart-increase-quota-portal/quotas-portal.png)

En la página de Resumen, selecciona un proveedor, como Compute o AML.

**Note** Para todos los proveedores excepto Compute, verás una columna Solicitar aumento en lugar de la columna Ajustable descrita abajo. Allí puedes solicitar un aumento para una cuota específica o crear una solicitud de soporte para el aumento.

En la página Mis cuotas, bajo Nombre de cuota, selecciona la cuota que deseas aumentar. Asegúrate de que la columna Ajustable muestre Sí para esta cuota.

Cerca de la parte superior de la página, selecciona Nueva solicitud de cuota, luego selecciona Ingresar un nuevo límite.

![Increase Quota](https://learn.microsoft.com/azure/quotas/media/quickstart-increase-quota-portal/enter-new-quota-limit.png)

En el panel Nueva solicitud de cuota, ingresa un valor numérico para tu nuevo límite de cuota y luego selecciona Enviar.

Tu solicitud será revisada y se te notificará si puede ser aprobada. Esto generalmente ocurre en pocos minutos.

Si tu solicitud no es aprobada, verás un enlace para crear una solicitud de soporte. Al usar este enlace, un ingeniero de soporte te asistirá con tu solicitud de aumento.

## Sugerencias de SKU para máquinas GPU de Azure Compute

[ND A100 v4-series](https://learn.microsoft.com/azure/virtual-machines/nda100-v4-series)

[ND H100 v5-series](https://learn.microsoft.com/azure/virtual-machines/nd-h100-v5-series)

[Standard_ND40rs_v2](https://learn.microsoft.com/azure/virtual-machines/ndv2-series)

Aquí algunos ejemplos:

### Si tienes GPUs A100 o H100

El ajuste fino completo suele ofrecer el mejor rendimiento. Puedes usar el siguiente comando para ajustar finamente Phi-3-V en la clasificación de memes de odio.

```bash
torchrun --nproc_per_node=8 --nnodes=<num_nodes> \
  --master_addr=$MASTER_ADDR --master_port=$MASTER_PORT --node_rank=$NODE_RANK \
  finetune_hf_trainer_hateful_memes.py \
  --output_dir <output_dir> \
  --batch_size 64 \
  --use_flash_attention \
  --bf16
```

### Si tienes Standard_ND40rs_v2 con 8x V100-32GB GPUs

Todavía es posible ajustar finamente Phi-3-V en la clasificación de memes de odio. Sin embargo, espera  
un rendimiento mucho menor comparado con GPUs A100 o H100 debido a la falta de soporte para flash attention.  
La precisión también podría verse afectada por la falta de soporte para bf16 (en su lugar se usa entrenamiento de precisión mixta fp16).

```bash
torchrun --nproc_per_node=8 --nnodes=<num_nodes> \
  --master_addr=$MASTER_ADDR --master_port=$MASTER_PORT --node_rank=$NODE_RANK \
  finetune_hf_trainer_hateful_memes.py \
  --output_dir <output_dir> \
  --batch_size 64
```

### Si no tienes acceso a GPUs de centro de datos

LoRA podría ser tu única opción. Puedes usar el siguiente comando para ajustar finamente Phi-3-V en la clasificación de memes de odio.

```bash
torchrun --nproc_per_node=2 \
  finetune_hf_trainer_hateful_memes.py \
  --output_dir <output_dir> \
  --batch_size 64 \
  --use_lora
```

Para GPUs Turing+ se soporta QLoRA

```bash
torchrun --nproc_per_node=2 \
  finetune_hf_trainer_hateful_memes.py \
  --output_dir <output_dir> \
  --batch_size 64 \
  --use_lora \
  --use_qlora
```

## Hiperparámetros sugeridos y precisión esperada

### NLVR2

```bash
torchrun --nproc_per_node=4 \
  finetune_hf_trainer_nlvr2.py \
  --bf16 --use_flash_attention \
  --batch_size 64 \
  --output_dir <output_dir> \
  --learning_rate <lr> \
  --num_train_epochs <epochs>

```

Método de entrenamiento | Modelo de visión congelado | tipo de dato | rango LoRA | alpha LoRA | tamaño de lote | tasa de aprendizaje | épocas | Precisión
--- | --- | --- | --- | --- | --- | --- | --- | --- |
ajuste fino completo |  | bf16 | - | - | 64 | 1e-5 | 3 | 89.40 |
ajuste fino completo | ✔ | bf16 | - | - | 64 | 2e-5 | 2 | 89.20 |
Resultados LoRA próximamente |  |  |  |  |  |  |  |  |

### NOTE  
Los resultados siguientes de DocVQA y memes de odio están basados en la versión anterior (Phi-3-vision).  
Los nuevos resultados con Phi-3.5-vision se actualizarán pronto.

### DocVQA (NOTE: Phi-3-vision)

```bash
torchrun --nproc_per_node=4 \
  finetune_hf_trainer_docvqa.py \
  --full_train \
  --bf16 --use_flash_attention \
  --batch_size 64 \
  --output_dir <output_dir> \
  --learning_rate <lr> \
  --num_train_epochs <epochs>

```

Método de entrenamiento | tipo de dato | rango LoRA | alpha LoRA | tamaño de lote | tasa de aprendizaje | épocas | ANLS
--- | --- | --- | --- | --- | --- | --- | --- |
ajuste fino completo | bf16 | - | - | 64 | 5e-6 | 2 | 83.65 |
ajuste fino completo | fp16 | - | - | 64 | 5e-6 | 2 | 82.60 |
modelo de imagen congelado | bf16 | - | - | 64 | 1e-4 | 2 | 79.19 |
modelo de imagen congelado | fp16 | - | - | 64 | 1e-4 | 2 | 78.74 |
LoRA | bf16 | 32 | 16 | 64 | 2e-4 | 2 | 82.46 |
LoRA | fp16 | 32 | 16 | 64 | 2e-4 | 2 | 82.34 |
QLoRA | bf16 | 32 | 16 | 64 | 2e-4 | 2 | 81.85 |
QLoRA | fp16 | 32 | 16 | 64 | 2e-4 | 2 | 81.85 |

### Memes de odio (NOTE: Phi-3-vision)

```bash
torchrun --nproc_per_node=4 \
  finetune_hf_trainer_hateful_memes.py \
  --bf16 --use_flash_attention \
  --batch_size 64 \
  --output_dir <output_dir> \
  --learning_rate <lr> \
  --num_train_epochs <epochs>

```

Método de entrenamiento | tipo de dato | rango LoRA | alpha LoRA | tamaño de lote | tasa de aprendizaje | épocas | Precisión
--- | --- | --- | --- | --- | --- | --- | --- |
ajuste fino completo | bf16 | - | - | 64 | 5e-5 | 2 | 86.4 |
ajuste fino completo | fp16 | - | - | 64 | 5e-5 | 2 | 85.4 |
modelo de imagen congelado | bf16 | - | - | 64 | 1e-4 | 3 | 79.4 |
modelo de imagen congelado | fp16 | - | - | 64 | 1e-4 | 3 | 78.6 |
LoRA | bf16 | 128 | 256 | 64 | 2e-4 | 2 | 86.6 |
LoRA | fp16 | 128 | 256 | 64 | 2e-4 | 2 | 85.2 |
QLoRA | bf16 | 128 | 256 | 64 | 2e-4 | 2 | 84.0 |
QLoRA | fp16 | 128 | 256 | 64 | 2e-4 | 2 | 83.8 |

## Benchmark de velocidad (NOTE: Phi-3-vision)

Los nuevos resultados de benchmark con Phi-3.5-vision se actualizarán pronto.

El benchmark de velocidad se realiza sobre el conjunto de datos DocVQA. La longitud promedio de secuencia de este conjunto  
es de 2443.23 tokens (usando `num_crops=16` para el modelo de imagen).

### 8x A100-80GB (Ampere)

Método de entrenamiento | \# nodos | GPUs | flash attention | Tamaño efectivo de lote | Rendimiento (img/s) | Aceleración | Memoria máxima GPU (GB)
--- | --- | --- | --- | --- | --- | --- | --- |
ajuste fino completo | 1 | 8 |  | 64 | 5.041 |  1x | ~42
ajuste fino completo | 1 | 8 | ✔ | 64 | 8.657 | 1.72x | ~36
ajuste fino completo | 2 | 16 | ✔ | 64 | 16.903 | 3.35x | ~29
ajuste fino completo | 4 | 32 | ✔ | 64 | 33.433 | 6.63x | ~26
modelo de imagen congelado | 1 | 8 |  | 64 | 17.578 | 3.49x | ~29
modelo de imagen congelado | 1 | 8 | ✔ | 64 | 31.736 | 6.30x | ~27
LoRA | 1 | 8 |  | 64 | 5.591 | 1.11x | ~50
LoRA | 1 | 8 | ✔ | 64 | 12.127 | 2.41x | ~16
QLoRA | 1 | 8 |  | 64 | 4.831 | 0.96x | ~32
QLoRA | 1 | 8 | ✔ | 64 | 10.545 | 2.09x | ~10

### 8x V100-32GB (Volta)

Método de entrenamiento | \# nodos | GPUs | flash attention | Tamaño efectivo de lote | Rendimiento (img/s) | Aceleración | Memoria máxima GPU (GB)
--- | --- | --- | --- | --- | --- | --- | --- |
ajuste fino completo | 1 | 8 | | 64 | 2.462 |  1x | ~32
ajuste fino completo | 2 | 16 |  | 64 | 4.182 | 1.70x | ~32
ajuste fino completo | 4 | 32 |  | 64 | 5.465 | 2.22x | ~32
modelo de imagen congelado | 1 | 8 |  | 64 | 8.942 | 3.63x | ~27
LoRA | 1 | 8 |  | 64 | 2.807 | 1.14x | ~30

## Problemas conocidos

- No se puede usar flash attention con fp16 (se recomienda siempre bf16 cuando esté disponible, y todas las GPUs que soportan flash attention también soportan bf16).  
- Aún no se soporta guardar puntos de control intermedios ni reanudar el entrenamiento.

**Aviso legal**:  
Este documento ha sido traducido utilizando el servicio de traducción automática [Co-op Translator](https://github.com/Azure/co-op-translator). Aunque nos esforzamos por la precisión, tenga en cuenta que las traducciones automáticas pueden contener errores o inexactitudes. El documento original en su idioma nativo debe considerarse la fuente autorizada. Para información crítica, se recomienda la traducción profesional realizada por humanos. No nos hacemos responsables de malentendidos o interpretaciones erróneas derivadas del uso de esta traducción.