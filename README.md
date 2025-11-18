import telebot
from telebot import types
import requests
import time

token = '8439716075:AAGVSe4UEwXNReeA1iFmJdSwzpt3Zr5yaIg'  # ⚠️ غير هذا التوكن!
bot = telebot.TeleBot(token)

@bot.message_handler(commands=['start'])
def start(message):
    btn1 = types.InlineKeyboardButton('• معلومات التوكن •', callback_data='btn1')
    btn2 = types.InlineKeyboardButton('• المطور •', url='https://t.me/RLH55')
    k = types.InlineKeyboardMarkup(row_width=1)
    k.add(btn1, btn2)
    bot.send_message(message.chat.id, """👋🏻
—————————————————
اهلاً بك عزيزي 
في بوت معلومات التوكن ❤""", reply_markup=k)  # إزالة parse_mode='html'

@bot.callback_query_handler(func=lambda call: True)
def Karar(call):
    if call.data == 'btn1':
        msg = bot.send_message(call.message.chat.id, "ارسل التوكن الان:")
        bot.register_next_step_handler(msg, nm)

def nm(message):
    token = message.text.strip()
    
    # تنظيف التوكن من المسافات
    token = token.replace(' ', '')
    
    # التحقق من صحة شكل التوكن
    if not token or ':' not in token:
        bot.send_message(message.chat.id, "❌ صيغة التوكن غير صحيحة.")
        return
    
    # إظهار أن البوت يعمل
    processing_msg = bot.send_message(message.chat.id, "⏳ جاري فحص التوكن...")
    
    try:
        # إضافة timeout لمنع الانتظار الطويل
        getme = requests.get(f"https://api.telegram.org/bot{token}/getMe", timeout=10).json()
        
        if not getme.get("ok"):
            error_desc = getme.get('description', 'خطأ غير معروف')
            bot.edit_message_text(f"❌ التوكن غير صالح:\n{error_desc}", 
                                message.chat.id, processing_msg.message_id)
            return

        # جلب معلومات الويب هوك
        webhook = requests.get(f"https://api.telegram.org/bot{token}/getWebhookInfo", timeout=10).json()

        user = getme["result"].get("username", "لا يوجد")
        name = getme["result"].get("first_name", "لا يوجد")
        user_id = getme["result"]["id"]
        webhook_url = webhook["result"].get("url", "❌ لا يوجد ويبهوك")
        
        # تنظيف البيانات من الرموز التي تسبب مشاكل في HTML
        name = str(name).replace('<', '').replace('>', '')
        user = str(user).replace('<', '').replace('>', '')
        webhook_url = str(webhook_url).replace('<', '').replace('>', '')

        btn = types.InlineKeyboardButton('• المطور •', url='https://t.me/RLH55')
        c = types.InlineKeyboardMarkup(row_width=1)
        c.add(btn)

        # إرسال النتيجة بدون HTML
        result_text = f"""
✅ معلومات التوكن
——————————————
👤 الاسم: {name}
📎 اليوزر: @{user}
🆔 الايدي: {user_id}
🌐 الويبهوك: {webhook_url}
        """
        
        bot.edit_message_text(result_text, 
                            message.chat.id, processing_msg.message_id,
                            reply_markup=c)

    except requests.exceptions.Timeout:
        bot.edit_message_text("❌ انتهت مهلة الاتصال. حاول مرة أخرى.",
                            message.chat.id, processing_msg.message_id)
    except requests.exceptions.ConnectionError:
        bot.edit_message_text("❌ خطأ في الاتصال بالإنترنت.",
                            message.chat.id, processing_msg.message_id)
    except Exception as e:
        bot.edit_message_text(f"❌ حدث خطأ:\n{str(e)}",
                            message.chat.id, processing_msg.message_id)

bot.polling()
