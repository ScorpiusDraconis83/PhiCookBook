<!--
CO_OP_TRANSLATOR_METADATA:
{
  "original_hash": "a8de701a2f1eb12b1f82432288d709cf",
  "translation_date": "2025-07-17T04:53:21+00:00",
  "source_file": "md/02.Application/04.Vision/Phi3/E2E_Nvidia_NIM_Vision.md",
  "language_code": "hk"
}
-->
### 範例情境

假設你有一張圖片（`demo.png`），並且想要產生 Python 程式碼來處理這張圖片，並儲存一個新的版本（`phi-3-vision.jpg`）。

上面的程式碼自動化了這個流程，步驟如下：

1. 設定環境和必要的配置。
2. 建立一個提示，指示模型產生所需的 Python 程式碼。
3. 將提示發送給模型並收集產生的程式碼。
4. 擷取並執行產生的程式碼。
5. 顯示原始圖片和處理後的圖片。

這種方法利用 AI 的力量來自動化影像處理任務，讓你更輕鬆且更快速地達成目標。

[Sample Code Solution](../../../../../../code/06.E2E/E2E_Nvidia_NIM_Phi3_Vision.ipynb)

讓我們一步步拆解整段程式碼的功能：

1. **安裝所需套件**：
    ```python
    !pip install langchain_nvidia_ai_endpoints -U
    ```
    這個指令會安裝 `langchain_nvidia_ai_endpoints` 套件，並確保是最新版本。

2. **匯入必要模組**：
    ```python
    from langchain_nvidia_ai_endpoints import ChatNVIDIA
    import getpass
    import os
    import base64
    ```
    這些匯入語句引入了與 NVIDIA AI 端點互動、密碼安全處理、作業系統操作，以及 base64 編碼/解碼相關的模組。

3. **設定 API 金鑰**：
    ```python
    if not os.getenv("NVIDIA_API_KEY"):
        os.environ["NVIDIA_API_KEY"] = getpass.getpass("Enter your NVIDIA API key: ")
    ```
    這段程式碼會檢查 `NVIDIA_API_KEY` 環境變數是否已設定，若沒有，會安全地提示使用者輸入 API 金鑰。

4. **定義模型與圖片路徑**：
    ```python
    model = 'microsoft/phi-3-vision-128k-instruct'
    chat = ChatNVIDIA(model=model)
    img_path = './imgs/demo.png'
    ```
    這裡設定要使用的模型，建立指定模型的 `ChatNVIDIA` 實例，並定義圖片檔案的路徑。

5. **建立文字提示**：
    ```python
    text = "Please create Python code for image, and use plt to save the new picture under imgs/ and name it phi-3-vision.jpg."
    ```
    這段定義了一個文字提示，指示模型產生用於處理圖片的 Python 程式碼。

6. **將圖片編碼成 Base64**：
    ```python
    with open(img_path, "rb") as f:
        image_b64 = base64.b64encode(f.read()).decode()
    image = f'<img src="data:image/png;base64,{image_b64}" />'
    ```
    這段程式碼讀取圖片檔案，將其編碼成 base64，並建立一個包含編碼資料的 HTML 圖片標籤。

7. **將文字與圖片結合成提示**：
    ```python
    prompt = f"{text} {image}"
    ```
    這裡將文字提示和 HTML 圖片標籤合併成一個字串。

8. **使用 ChatNVIDIA 產生程式碼**：
    ```python
    code = ""
    for chunk in chat.stream(prompt):
        print(chunk.content, end="")
        code += chunk.content
    ```
    這段程式碼將提示發送給 `ChatNVIDIA` 模型，並分段收集產生的程式碼，同時印出並附加每段內容到 `code` 字串。

9. **從產生內容中擷取 Python 程式碼**：
    ```python
    begin = code.index('```python') + 9
    code = code[begin:]
    end = code.index('```')
    code = code[:end]
    ```
    這段程式碼透過移除 markdown 格式，擷取出真正的 Python 程式碼。

10. **執行產生的程式碼**：
    ```python
    import subprocess
    result = subprocess.run(["python", "-c", code], capture_output=True)
    ```
    這裡以子程序方式執行擷取出的 Python 程式碼，並捕捉其輸出。

11. **顯示圖片**：
    ```python
    from IPython.display import Image, display
    display(Image(filename='./imgs/phi-3-vision.jpg'))
    display(Image(filename='./imgs/demo.png'))
    ```
    這些程式碼使用 `IPython.display` 模組來顯示圖片。

**免責聲明**：  
本文件由 AI 翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 進行翻譯。雖然我們致力於確保準確性，但請注意自動翻譯可能包含錯誤或不準確之處。原始文件的母語版本應被視為權威來源。對於重要資訊，建議採用專業人工翻譯。我們不對因使用本翻譯而引起的任何誤解或誤釋承擔責任。