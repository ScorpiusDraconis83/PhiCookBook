<!--
CO_OP_TRANSLATOR_METADATA:
{
  "original_hash": "d658062de70b131ef4c0bff69b5fc70e",
  "translation_date": "2025-07-16T21:42:09+00:00",
  "source_file": "md/01.Introduction/04/QuantifyingPhi.md",
  "language_code": "fa"
}
-->
# **کوانتایز کردن خانواده فی**

کوانتایز کردن مدل به فرآیند نگاشت پارامترها (مانند وزن‌ها و مقادیر فعال‌سازی) در یک مدل شبکه عصبی از یک بازه بزرگ مقادیر (معمولاً بازه‌ای پیوسته) به یک بازه محدود و کوچک‌تر گفته می‌شود. این فناوری می‌تواند اندازه و پیچیدگی محاسباتی مدل را کاهش داده و کارایی اجرای مدل را در محیط‌های با منابع محدود مانند دستگاه‌های موبایل یا سیستم‌های تعبیه‌شده بهبود بخشد. کوانتایز کردن مدل با کاهش دقت پارامترها فشرده‌سازی را محقق می‌کند، اما در عین حال مقداری افت دقت نیز به همراه دارد. بنابراین، در فرآیند کوانتایز کردن باید تعادلی بین اندازه مدل، پیچیدگی محاسباتی و دقت برقرار شود. روش‌های رایج کوانتایز کردن شامل کوانتایز کردن نقطه ثابت، کوانتایز کردن نقطه شناور و غیره هستند. شما می‌توانید بر اساس سناریو و نیازهای خاص، استراتژی کوانتایز مناسب را انتخاب کنید.

ما امیدواریم مدل‌های GenAI را روی دستگاه‌های لبه‌ای مستقر کنیم و اجازه دهیم دستگاه‌های بیشتری وارد سناریوهای GenAI شوند، مانند دستگاه‌های موبایل، AI PC/Copilot+PC و دستگاه‌های سنتی IoT. از طریق مدل کوانتایز شده، می‌توانیم آن را بر اساس دستگاه‌های مختلف روی دستگاه‌های لبه‌ای مختلف مستقر کنیم. با ترکیب چارچوب تسریع مدل و مدل کوانتایز شده ارائه شده توسط تولیدکنندگان سخت‌افزار، می‌توانیم سناریوهای کاربردی SLM بهتری بسازیم.

در سناریوی کوانتایز کردن، دقت‌های مختلفی داریم (INT4، INT8، FP16، FP32). در ادامه توضیحاتی درباره دقت‌های رایج کوانتایز ارائه شده است.

### **INT4**

کوانتایز کردن INT4 روشی بسیار فشرده است که وزن‌ها و مقادیر فعال‌سازی مدل را به اعداد صحیح ۴ بیتی تبدیل می‌کند. کوانتایز INT4 معمولاً به دلیل بازه نمایشی کوچک‌تر و دقت پایین‌تر، باعث افت دقت بیشتری می‌شود. با این حال، در مقایسه با کوانتایز INT8، کوانتایز INT4 می‌تواند نیازهای ذخیره‌سازی و پیچیدگی محاسباتی مدل را بیشتر کاهش دهد. لازم به ذکر است که کوانتایز INT4 در کاربردهای عملی نسبتاً نادر است، زیرا دقت بسیار پایین ممکن است باعث کاهش قابل توجه عملکرد مدل شود. علاوه بر این، همه سخت‌افزارها از عملیات INT4 پشتیبانی نمی‌کنند، بنابراین هنگام انتخاب روش کوانتایز باید سازگاری سخت‌افزاری را در نظر گرفت.

### **INT8**

کوانتایز INT8 فرآیند تبدیل وزن‌ها و مقادیر فعال‌سازی مدل از اعداد نقطه شناور به اعداد صحیح ۸ بیتی است. اگرچه بازه عددی نمایش داده شده توسط اعداد صحیح INT8 کوچکتر و دقت کمتری دارد، اما می‌تواند به طور قابل توجهی نیازهای ذخیره‌سازی و محاسبات را کاهش دهد. در کوانتایز INT8، وزن‌ها و مقادیر فعال‌سازی مدل از طریق فرآیند کوانتایز شامل مقیاس‌بندی و جابجایی عبور می‌کنند تا اطلاعات اصلی نقطه شناور تا حد امکان حفظ شود. در زمان استنتاج، این مقادیر کوانتایز شده به اعداد نقطه شناور بازگردانده شده و سپس برای مرحله بعدی دوباره به INT8 کوانتایز می‌شوند. این روش می‌تواند دقت کافی را در اکثر کاربردها فراهم کند و در عین حال کارایی محاسباتی بالایی داشته باشد.

### **FP16**

فرمت FP16، یعنی اعداد نقطه شناور ۱۶ بیتی (float16)، نسبت به اعداد نقطه شناور ۳۲ بیتی (float32) حافظه را نصف می‌کند که در کاربردهای یادگیری عمیق بزرگ مقیاس مزایای قابل توجهی دارد. فرمت FP16 اجازه می‌دهد مدل‌های بزرگ‌تر بارگذاری شوند یا داده‌های بیشتری در محدودیت‌های حافظه GPU پردازش شوند. با ادامه پشتیبانی سخت‌افزارهای GPU مدرن از عملیات FP16، استفاده از فرمت FP16 ممکن است باعث افزایش سرعت محاسبات نیز شود. با این حال، فرمت FP16 معایب ذاتی خود را دارد، یعنی دقت پایین‌تر که ممکن است در برخی موارد منجر به ناپایداری عددی یا از دست رفتن دقت شود.

### **FP32**

فرمت FP32 دقت بالاتری ارائه می‌دهد و می‌تواند بازه وسیعی از مقادیر را به دقت نمایش دهد. در سناریوهایی که عملیات ریاضی پیچیده انجام می‌شود یا نتایج با دقت بالا مورد نیاز است، فرمت FP32 ترجیح داده می‌شود. اما دقت بالا به معنای مصرف بیشتر حافظه و زمان محاسبه طولانی‌تر است. برای مدل‌های یادگیری عمیق بزرگ مقیاس، به ویژه زمانی که پارامترهای مدل زیاد و داده‌ها حجیم هستند، فرمت FP32 ممکن است باعث کمبود حافظه GPU یا کاهش سرعت استنتاج شود.

در دستگاه‌های موبایل یا دستگاه‌های IoT، می‌توانیم مدل‌های Phi-3.x را به INT4 تبدیل کنیم، در حالی که AI PC / Copilot PC می‌توانند از دقت‌های بالاتری مانند INT8، FP16، FP32 استفاده کنند.

در حال حاضر، تولیدکنندگان سخت‌افزار مختلف چارچوب‌های متفاوتی برای پشتیبانی از مدل‌های مولد دارند، مانند OpenVINO اینتل، QNN کوالکام، MLX اپل و CUDA انویدیا و غیره که همراه با کوانتایز کردن مدل، استقرار محلی را کامل می‌کنند.

از نظر فناوری، پس از کوانتایز کردن، فرمت‌های مختلفی پشتیبانی می‌شوند، مانند فرمت PyTorch / Tensorflow، GGUF و ONNX. من مقایسه‌ای بین فرمت‌های GGUF و ONNX و سناریوهای کاربردی آن‌ها انجام داده‌ام. در اینجا فرمت کوانتایز ONNX را توصیه می‌کنم که از چارچوب مدل تا سخت‌افزار پشتیبانی خوبی دارد. در این فصل، تمرکز ما بر ONNX Runtime برای GenAI، OpenVINO و Apple MLX برای انجام کوانتایز کردن مدل خواهد بود (اگر راه بهتری دارید، می‌توانید با ارسال PR به ما ارائه دهید).

**این فصل شامل موارد زیر است**

1. [کوانتایز کردن Phi-3.5 / 4 با استفاده از llama.cpp](./UsingLlamacppQuantifyingPhi.md)

2. [کوانتایز کردن Phi-3.5 / 4 با استفاده از افزونه‌های Generative AI برای onnxruntime](./UsingORTGenAIQuantifyingPhi.md)

3. [کوانتایز کردن Phi-3.5 / 4 با استفاده از Intel OpenVINO](./UsingIntelOpenVINOQuantifyingPhi.md)

4. [کوانتایز کردن Phi-3.5 / 4 با استفاده از چارچوب Apple MLX](./UsingAppleMLXQuantifyingPhi.md)

**سلب مسئولیت**:  
این سند با استفاده از سرویس ترجمه هوش مصنوعی [Co-op Translator](https://github.com/Azure/co-op-translator) ترجمه شده است. در حالی که ما در تلاش برای دقت هستیم، لطفاً توجه داشته باشید که ترجمه‌های خودکار ممکن است حاوی خطاها یا نادرستی‌هایی باشند. سند اصلی به زبان بومی خود باید به عنوان منبع معتبر در نظر گرفته شود. برای اطلاعات حیاتی، ترجمه حرفه‌ای انسانی توصیه می‌شود. ما مسئول هیچ گونه سوءتفاهم یا تفسیر نادرستی که از استفاده این ترجمه ناشی شود، نیستیم.