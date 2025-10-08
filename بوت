# -*- coding: utf-8 -*-
import telebot
import requests
import json
import time
import io
import base64

# ==============================================================================
# 1. إعدادات البوت ومفتاح API (Bot and API Settings)
# ==============================================================================

# رمز البوت الخاص بك من BotFather (The token you provided)
TELEGRAM_TOKEN = "7308302669:AAFcw-Xm80Vg9hJOofNAr735RqknggxkHz0"

# مفتاح API الخاص بـ Gemini. نتركه فارغاً كما هو مطلوب، وسيتم توفيره في بيئة التشغيل.
GEMINI_API_KEY = "" 

# تهيئة البوت (Initialize the bot)
bot = telebot.TeleBot(TELEGRAM_TOKEN)

# ==============================================================================
# 2. دالة توليد الصورة باستخدام Imagen (Image Generation Function)
# ==============================================================================

def generate_image_from_prompt(prompt):
    """
    يتصل بواجهة برمجة تطبيقات Imagen 3.0 لتوليد صورة من نص.
    (Connects to Imagen 3.0 API to generate an image from text.)
    """
    
    # نقطة نهاية API لتوليد الصور (API endpoint for image generation)
    api_url = f"https://generativelanguage.googleapis.com/v1beta/models/imagen-3.0-generate-002:predict?key={GEMINI_API_KEY}"
    
    # الحمولة (Payload) المطلوبة للطلب
    payload = {
        "instances": {
            "prompt": prompt,
            # يمكنك تعديل الأبعاد (مثلاً "1024x1024") إذا أردت
            "imageSize": "1024x1024" 
        },
        "parameters": {
            "sampleCount": 1
        }
    }
    
    headers = {'Content-Type': 'application/json'}
    
    # تطبيق ميزة التراجع الأُسّي للمحاولات الفاشلة (Apply exponential backoff for retries)
    max_retries = 3
    for attempt in range(max_retries):
        try:
            # إرسال الطلب (Send the request)
            response = requests.post(api_url, headers=headers, data=json.dumps(payload), timeout=30)
            response.raise_for_status() # إلقاء خطأ إذا كان رمز الحالة 4xx أو 5xx

            # معالجة الاستجابة (Process the response)
            result = response.json()
            
            # التحقق من وجود بيانات الصورة (Check for image data)
            if (result.get('predictions') and 
                len(result['predictions']) > 0 and 
                result['predictions'][0].get('bytesBase64Encoded')):
                
                # استخراج بيانات الصورة المشفرة (Extract the encoded image data)
                base64_data = result['predictions'][0]['bytesBase64Encoded']
                
                # فك تشفير البيانات وإعادتها كبايت (Decode the data and return as bytes)
                return base64.b64decode(base64_data)
            else:
                print("API returned no image data.")
                return None

        except requests.exceptions.RequestException as e:
            print(f"Attempt {attempt + 1} failed: {e}")
            if attempt < max_retries - 1:
                # الانتظار قبل المحاولة التالية (Wait before the next attempt)
                wait_time = 2 ** attempt 
                time.sleep(wait_time)
            else:
                # فشلت جميع المحاولات (All attempts failed)
                return None
        except Exception as e:
            print(f"An unexpected error occurred: {e}")
            return None

    return None

# ==============================================================================
# 3. معالج رسائل التلجرام (Telegram Message Handler)
# ==============================================================================

@bot.message_handler(func=lambda message: True)
def handle_message(message):
    """
    يتعامل مع جميع الرسائل النصية من المستخدم.
    (Handles all text messages from the user.)
    """
    prompt = message.text.strip()
    chat_id = message.chat.id
    
    if not prompt or len(prompt) < 5:
        bot.reply_to(message, 
                     "من فضلك، أرسل وصفاً مفصلاً للصورة التي تريد تصميمها (5 أحرف على الأقل).")
        return

    # رسالة جاري المعالجة (Processing message)
    status_message = bot.send_message(chat_id, 
                                      "✨ جاري تصميم صورتك بالذكاء الاصطناعي... قد يستغرق هذا بضع ثوانٍ. 🎨")

    try:
        # توليد الصورة (Generate the image)
        image_bytes = generate_image_from_prompt(prompt)

        # حذف رسالة 'جاري المعالجة' (Delete the 'Processing' message)
        bot.delete_message(chat_id, status_message.message_id)

        if image_bytes:
            # إرسال الصورة إلى التلجرام (Send the image to Telegram)
            image_stream = io.BytesIO(image_bytes)
            image_stream.name = 'ai_generated_image.png'
            
            bot.send_photo(chat_id, 
                           image_stream, 
                           caption=f"✅ **تم التصميم بنجاح!**\n\n**الوصف:** *{prompt}*",
                           parse_mode='Markdown')
        else:
            bot.send_message(chat_id, 
                             "❌ عذراً، لم أتمكن من توليد الصورة. قد تكون هناك مشكلة في الوصف أو الاتصال بخدمة الذكاء الاصطناعي.")

    except Exception as e:
        print(f"Error handling message: {e}")
        bot.send_message(chat_id, 
                         "وقع خطأ غير متوقع أثناء المعالجة. يرجى المحاولة مرة أخرى لاحقاً.")

# ==============================================================================
# 4. بدء تشغيل البوت (Start the Bot)
# ==============================================================================

print("البوت يعمل الآن... اضغط Ctrl+C لإيقافه.")
# تشغيل البوت في وضع الاستقصاء (Polling mode)
bot.polling(non_stop=True, interval=0)
