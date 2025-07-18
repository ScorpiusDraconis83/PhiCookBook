<!--
CO_OP_TRANSLATOR_METADATA:
{
  "original_hash": "8ec74e4a49934dad78bc52dcb898359c",
  "translation_date": "2025-07-16T17:09:44+00:00",
  "source_file": "code/07.Lab/RAG_with_PromptFlow_and_AISearch/README.md",
  "language_code": "id"
}
-->
## RAG dengan PromptFlow dan AISearch

Dalam contoh ini, kita akan mengimplementasikan aplikasi Retrieval Augmented Generation (RAG) dengan memanfaatkan Phi3 sebagai SLM, AI Search sebagai vectorDB, dan Prompt Flow sebagai low-code orchestrator.

## Fitur

- Mudah diterapkan menggunakan Docker.  
- Arsitektur yang dapat diskalakan untuk menangani alur kerja AI.  
- Pendekatan low code menggunakan Prompt Flow  

## Prasyarat

Sebelum memulai, pastikan Anda telah memenuhi persyaratan berikut:

- Docker terpasang di mesin lokal Anda.  
- Akun Azure dengan izin untuk membuat dan mengelola sumber daya container.  
- Instance Azure AI Studio dan Azure AI Search  
- Model embedding untuk membuat indeks Anda (bisa menggunakan embedding Azure OpenAI atau model OS dari katalog)  
- Python 3.8 atau versi lebih baru terpasang di mesin lokal Anda.  
- Azure Container Registry (atau registry lain sesuai pilihan Anda)  

## Instalasi

1. Buat flow baru di proyek Azure AI Studio Anda menggunakan file flow.yaml.  
2. Deploy Model Phi3 dari katalog model Azure AI Anda dan buat koneksi ke proyek Anda. [Deploy Phi-3 as a Model as a Service](https://learn.microsoft.com/azure/machine-learning/how-to-deploy-models-phi-3?view=azureml-api-2&tabs=phi-3-mini)  
3. Buat indeks vektor di Azure AI Search menggunakan dokumen pilihan Anda [Create a vector index on Azure AI Search](https://learn.microsoft.com/azure/search/search-how-to-create-search-index?tabs=portal)  
4. Deploy flow pada endpoint terkelola dan gunakan di file prompt-flow-frontend.py. [Deploy a flow on an online endpoint](https://learn.microsoft.com/azure/ai-studio/how-to/flow-deploy)  
5. Clone repository:

    ```sh
    git clone [[https://github.com/yourusername/prompt-flow-frontend.git](https://github.com/microsoft/Phi-3CookBook.git)](https://github.com/microsoft/Phi-3CookBook.git)
    
    cd code/07.Lab/RAG with PromptFlow and AISearch
    ```

6. Bangun image Docker:

    ```sh
    docker build -t prompt-flow-frontend.py .
    ```

7. Push image Docker ke Azure Container Registry:

    ```sh
    az acr login --name yourregistry
    
    docker tag prompt-flow-frontend.py:latest yourregistry.azurecr.io/prompt-flow-frontend.py:latest
    
    docker push yourregistry.azurecr.io/prompt-flow-frontend.py:latest
    ```

## Penggunaan

1. Jalankan container Docker:

    ```sh
    docker run -p 8501:8501 yourregistry.azurecr.io/prompt-flow-frontend.py:latest
    ```

2. Akses aplikasi melalui browser Anda di `http://localhost:8501`.

## Kontak

Valentina Alto - [Linkedin](https://www.linkedin.com/in/valentina-alto-6a0590148/)

Artikel Lengkap: [RAG with Phi-3-Medium as a Model as a Service from Azure Model Catalog](https://medium.com/@valentinaalto/rag-with-phi-3-medium-as-a-model-as-a-service-from-azure-model-catalog-62e1411948f3)

**Penafian**:  
Dokumen ini telah diterjemahkan menggunakan layanan terjemahan AI [Co-op Translator](https://github.com/Azure/co-op-translator). Meskipun kami berupaya untuk mencapai akurasi, harap diperhatikan bahwa terjemahan otomatis mungkin mengandung kesalahan atau ketidakakuratan. Dokumen asli dalam bahasa aslinya harus dianggap sebagai sumber yang sahih. Untuk informasi penting, disarankan menggunakan terjemahan profesional oleh manusia. Kami tidak bertanggung jawab atas kesalahpahaman atau penafsiran yang keliru yang timbul dari penggunaan terjemahan ini.