<!--
CO_OP_TRANSLATOR_METADATA:
{
  "original_hash": "a8de701a2f1eb12b1f82432288d709cf",
  "translation_date": "2025-07-17T04:52:41+00:00",
  "source_file": "md/02.Application/04.Vision/Phi3/E2E_Nvidia_NIM_Vision.md",
  "language_code": "fa"
}
-->
### سناریوی نمونه

تصور کنید یک تصویر (`demo.png`) دارید و می‌خواهید کدی به زبان Python تولید کنید که این تصویر را پردازش کرده و نسخه جدیدی از آن را ذخیره کند (`phi-3-vision.jpg`).

کد بالا این فرایند را به صورت خودکار انجام می‌دهد با:

1. راه‌اندازی محیط و تنظیمات لازم.
2. ایجاد یک متن راهنما که مدل را برای تولید کد Python مورد نیاز هدایت می‌کند.
3. ارسال متن راهنما به مدل و جمع‌آوری کد تولید شده.
4. استخراج و اجرای کد تولید شده.
5. نمایش تصاویر اصلی و پردازش شده.

این روش از قدرت هوش مصنوعی برای خودکارسازی وظایف پردازش تصویر استفاده می‌کند و رسیدن به اهداف شما را آسان‌تر و سریع‌تر می‌کند.

[نمونه کد حل مسئله](../../../../../../code/06.E2E/E2E_Nvidia_NIM_Phi3_Vision.ipynb)

بیایید قدم به قدم ببینیم کل کد چه کاری انجام می‌دهد:

1. **نصب بسته مورد نیاز**:
    ```python
    !pip install langchain_nvidia_ai_endpoints -U
    ```
    این دستور بسته `langchain_nvidia_ai_endpoints` را نصب می‌کند و مطمئن می‌شود که آخرین نسخه آن استفاده می‌شود.

2. **وارد کردن ماژول‌های لازم**:
    ```python
    from langchain_nvidia_ai_endpoints import ChatNVIDIA
    import getpass
    import os
    import base64
    ```
    این واردسازی‌ها ماژول‌های لازم برای تعامل با نقاط پایانی NVIDIA AI، مدیریت امن رمز عبور، تعامل با سیستم عامل و رمزگذاری/رمزگشایی داده‌ها به فرمت base64 را فراهم می‌کنند.

3. **تنظیم کلید API**:
    ```python
    if not os.getenv("NVIDIA_API_KEY"):
        os.environ["NVIDIA_API_KEY"] = getpass.getpass("Enter your NVIDIA API key: ")
    ```
    این کد بررسی می‌کند که آیا متغیر محیطی `NVIDIA_API_KEY` تنظیم شده است یا خیر. اگر تنظیم نشده باشد، از کاربر می‌خواهد کلید API خود را به صورت امن وارد کند.

4. **تعریف مدل و مسیر تصویر**:
    ```python
    model = 'microsoft/phi-3-vision-128k-instruct'
    chat = ChatNVIDIA(model=model)
    img_path = './imgs/demo.png'
    ```
    این بخش مدل مورد استفاده را تعیین می‌کند، نمونه‌ای از `ChatNVIDIA` با مدل مشخص شده ایجاد می‌کند و مسیر فایل تصویر را تعریف می‌کند.

5. **ایجاد متن راهنما**:
    ```python
    text = "Please create Python code for image, and use plt to save the new picture under imgs/ and name it phi-3-vision.jpg."
    ```
    این متن راهنما مدلی را هدایت می‌کند تا کد Python برای پردازش تصویر تولید کند.

6. **رمزگذاری تصویر به صورت Base64**:
    ```python
    with open(img_path, "rb") as f:
        image_b64 = base64.b64encode(f.read()).decode()
    image = f'<img src="data:image/png;base64,{image_b64}" />'
    ```
    این کد فایل تصویر را می‌خواند، آن را به base64 رمزگذاری می‌کند و یک تگ HTML تصویر با داده‌های رمزگذاری شده ایجاد می‌کند.

7. **ترکیب متن و تصویر در یک راهنما**:
    ```python
    prompt = f"{text} {image}"
    ```
    این بخش متن راهنما و تگ HTML تصویر را در یک رشته واحد ترکیب می‌کند.

8. **تولید کد با استفاده از ChatNVIDIA**:
    ```python
    code = ""
    for chunk in chat.stream(prompt):
        print(chunk.content, end="")
        code += chunk.content
    ```
    این کد متن راهنما را به مدل `ChatNVIDIA` ارسال می‌کند و کد تولید شده را به صورت بخش بخش دریافت کرده، هر بخش را چاپ و به رشته `code` اضافه می‌کند.

9. **استخراج کد Python از محتوای تولید شده**:
    ```python
    begin = code.index('```python') + 9
    code = code[begin:]
    end = code.index('```')
    code = code[:end]
    ```
    این بخش کد واقعی Python را از محتوای تولید شده استخراج می‌کند و قالب‌بندی markdown را حذف می‌کند.

10. **اجرای کد تولید شده**:
    ```python
    import subprocess
    result = subprocess.run(["python", "-c", code], capture_output=True)
    ```
    این کد استخراج شده را به عنوان یک فرایند فرعی اجرا می‌کند و خروجی آن را دریافت می‌کند.

11. **نمایش تصاویر**:
    ```python
    from IPython.display import Image, display
    display(Image(filename='./imgs/phi-3-vision.jpg'))
    display(Image(filename='./imgs/demo.png'))
    ```
    این خطوط تصاویر را با استفاده از ماژول `IPython.display` نمایش می‌دهند.

**سلب مسئولیت**:  
این سند با استفاده از سرویس ترجمه هوش مصنوعی [Co-op Translator](https://github.com/Azure/co-op-translator) ترجمه شده است. در حالی که ما در تلاش برای دقت هستیم، لطفاً توجه داشته باشید که ترجمه‌های خودکار ممکن است حاوی خطاها یا نواقصی باشند. سند اصلی به زبان بومی خود باید به عنوان منبع معتبر در نظر گرفته شود. برای اطلاعات حیاتی، ترجمه حرفه‌ای انسانی توصیه می‌شود. ما مسئول هیچ گونه سوءتفاهم یا تفسیر نادرستی که از استفاده این ترجمه ناشی شود، نیستیم.