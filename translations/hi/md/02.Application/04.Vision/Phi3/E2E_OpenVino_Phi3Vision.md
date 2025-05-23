<!--
CO_OP_TRANSLATOR_METADATA:
{
  "original_hash": "d7d7afa242a4a041ff4193546d4baf16",
  "translation_date": "2025-05-08T05:27:08+00:00",
  "source_file": "md/02.Application/04.Vision/Phi3/E2E_OpenVino_Phi3Vision.md",
  "language_code": "hi"
}
-->
यह डेमो दिखाता है कि एक pretrained मॉडल का उपयोग करके कैसे एक छवि और टेक्स्ट प्रॉम्प्ट के आधार पर Python कोड जनरेट किया जा सकता है।

[Sample Code](../../../../../../code/06.E2E/E2E_OpenVino_Phi3-vision.ipynb)

यहाँ चरण-दर-चरण व्याख्या दी गई है:

1. **इम्पोर्ट्स और सेटअप**:
   - आवश्यक लाइब्रेरी और मॉड्यूल इम्पोर्ट किए जाते हैं, जिनमें छवि प्रसंस्करण के लिए `requests`, `PIL` और मॉडल तथा प्रोसेसिंग के लिए `transformers` शामिल हैं।

2. **छवि लोड करना और दिखाना**:
   - एक छवि फ़ाइल (`demo.png`) `PIL` लाइब्रेरी का उपयोग करके खोली जाती है और प्रदर्शित की जाती है।

3. **प्रॉम्प्ट परिभाषित करना**:
   - एक संदेश तैयार किया जाता है जिसमें छवि शामिल होती है और Python कोड जनरेट करने का अनुरोध किया जाता है जो छवि को प्रोसेस करे और `plt` (matplotlib) का उपयोग करके उसे सेव करे।

4. **प्रोसेसर लोड करना**:
   - `AutoProcessor` को pretrained मॉडल से लोड किया जाता है, जिसे `out_dir` डायरेक्टरी द्वारा निर्दिष्ट किया गया है। यह प्रोसेसर टेक्स्ट और छवि इनपुट को संभालेगा।

5. **प्रॉम्प्ट बनाना**:
   - `apply_chat_template` मेथड का उपयोग करके संदेश को मॉडल के लिए उपयुक्त प्रॉम्प्ट में फॉर्मेट किया जाता है।

6. **इनपुट्स को प्रोसेस करना**:
   - प्रॉम्प्ट और छवि को ऐसे टेन्सर में बदला जाता है जिन्हें मॉडल समझ सके।

7. **जनरेशन आर्गुमेंट्स सेट करना**:
   - मॉडल के जनरेशन प्रोसेस के लिए आर्गुमेंट्स परिभाषित किए जाते हैं, जैसे अधिकतम नए टोकन की संख्या और आउटपुट को सैंपल करना है या नहीं।

8. **कोड जनरेट करना**:
   - मॉडल इनपुट्स और जनरेशन आर्गुमेंट्स के आधार पर Python कोड जनरेट करता है। आउटपुट को संभालने के लिए `TextStreamer` का उपयोग किया जाता है, जिसमें प्रॉम्प्ट और स्पेशल टोकन को स्किप किया जाता है।

9. **आउटपुट**:
   - जनरेट किया गया कोड प्रिंट किया जाता है, जिसमें वह Python कोड शामिल होता है जो छवि को प्रोसेस करता है और प्रॉम्प्ट में निर्दिष्ट अनुसार उसे सेव करता है।

यह डेमो दिखाता है कि OpenVino का उपयोग करके pretrained मॉडल की मदद से उपयोगकर्ता इनपुट और छवियों के आधार पर कोड को डायनामिक रूप से कैसे जनरेट किया जा सकता है।

**अस्वीकरण**:  
इस दस्तावेज़ का अनुवाद AI अनुवाद सेवा [Co-op Translator](https://github.com/Azure/co-op-translator) का उपयोग करके किया गया है। जबकि हम सटीकता के लिए प्रयासरत हैं, कृपया ध्यान रखें कि स्वचालित अनुवाद में त्रुटियाँ या अशुद्धियाँ हो सकती हैं। मूल दस्तावेज़ अपनी मूल भाषा में ही आधिकारिक स्रोत माना जाना चाहिए। महत्वपूर्ण जानकारी के लिए पेशेवर मानव अनुवाद की सिफारिश की जाती है। इस अनुवाद के उपयोग से उत्पन्न किसी भी गलतफहमी या गलत व्याख्या के लिए हम उत्तरदायी नहीं हैं।