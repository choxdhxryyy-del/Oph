import telebot
from telebot import types
from groq import Groq
import time
from dotenv import load_dotenv
import os

# 🔹 Load environment variables
load_dotenv()
TELEGRAM_TOKEN = os.getenv("7451412892:AAFMJcUiil0cPOFTQhW7x7BgbpWbuaKSrzg")
GROQ_API_KEY = os.getenv("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")

# 🔹 Initialize bot and Groq client
bot = telebot.TeleBot(TELEGRAM_TOKEN)
client = Groq(api_key=GROQ_API_KEY)

# 🔹 User stop flag
user_stop = {}

# 🔹 Helper: get username
def get_username(message):
    return f"@{message.from_user.username}" if message.from_user.username else message.from_user.first_name

# ===== /start command =====
@bot.message_handler(commands=['start'])
def start(message):
    chat_id = message.chat.id
    user_stop[chat_id] = False

    markup = types.InlineKeyboardMarkup(row_width=3)
    markup.add(
        types.InlineKeyboardButton("ℹ️ Help", callback_data="help"),
        types.InlineKeyboardButton("🤖 About", callback_data="about"),
        types.InlineKeyboardButton("❌ Stop", callback_data="stop")
    )

    username = get_username(message)
    bot.send_message(
        chat_id,
        f"👋 **Namaste {username}!**\n\n"
        "Main aapka professional AI assistant hoon.\n"
        "✅ Aap mujhse koi bhi sawal pooch sakte hain.\n\n"
        "Neeche diye gaye buttons se Help, About ya Stop choose karein.",
        parse_mode="Markdown",
        reply_markup=markup
    )

# ===== Callback buttons handler =====
@bot.callback_query_handler(func=lambda call: True)
def callback_query(call):
    chat_id = call.message.chat.id
    username = get_username(call.message)

    if call.data == "stop":
        user_stop[chat_id] = True
        markup = types.InlineKeyboardMarkup()
        markup.add(types.InlineKeyboardButton("🔄 Restart Bot", callback_data="start"))
        bot.edit_message_text(
            f"❌ **{username}, bot ab band ho gaya.**",
            chat_id=chat_id,
            message_id=call.message.message_id,
            parse_mode="Markdown",
            reply_markup=markup
        )

    elif call.data == "start":
        user_stop[chat_id] = False
        start(call.message)

    elif call.data == "help":
        bot.answer_callback_query(call.id)
        bot.send_message(
            chat_id,
            f"ℹ️ **{username},** mujhe koi bhi sawal bhejo aur mai AI ke through short reply dunga.",
            parse_mode="Markdown"
        )

    elif call.data == "about":
        bot.answer_callback_query(call.id)
        bot.send_message(
            chat_id,
            "🤖 **Professional AI Bot**\n\n"
            "Banaaya gaya Python aur Telegram API se.\n"
            "Groq AI (Llama 3.1) par based hai.\n"
            "Secure API key system use karta hai.",
            parse_mode="Markdown"
        )

# ===== /help, /about, /stop commands =====
@bot.message_handler(commands=['help'])
def help_command(message):
    username = get_username(message)
    bot.send_message(message.chat.id, f"ℹ️ **{username},** mujhe koi bhi sawal bhejo aur mai short reply dunga.", parse_mode="Markdown")

@bot.message_handler(commands=['about'])
def about_command(message):
    bot.send_message(
        message.chat.id,
        "🤖 **Professional AI Bot**\n\n"
        "Banaaya gaya Python aur Telegram API se.\n"
        "Groq AI (Llama 3.1) par based hai.\n"
        "Secure API key system use karta hai.",
        parse_mode="Markdown"
    )

@bot.message_handler(commands=['stop'])
def stop_command(message):
    chat_id = message.chat.id
    user_stop[chat_id] = True
    username = get_username(message)
    markup = types.InlineKeyboardMarkup()
    markup.add(types.InlineKeyboardButton("🔄 Restart Bot", callback_data="start"))
    bot.send_message(chat_id, f"❌ **{username}, bot ab band ho gaya.**", parse_mode="Markdown", reply_markup=markup)

# ===== Text handler =====
@bot.message_handler(func=lambda m: True)
def chat(message):
    chat_id = message.chat.id
    text = message.text.strip()
    username = get_username(message)

    if user_stop.get(chat_id, False):
        bot.send_message(chat_id, f"⚠️ **{username}, bot band hai.** Dubara chalu karne ke liye /start likho.", parse_mode="Markdown")
        return

    bot.send_chat_action(chat_id, "typing")
    time.sleep(1)

    try:
        # Groq API call with short/concise Hindi reply
        response = client.chat.completions.create(
            model="llama-3.1-8b-instant",
            messages=[{"role": "user", "content": f"Reply shortly and concisely in Hindi: {text}"}],
        )
        reply = response.choices[0].message.content.strip()
        bot.send_message(chat_id, f"💬 **{username},** {reply}", parse_mode="Markdown")

    except Exception as e:
        print(f"Groq API error: {e}")
        bot.send_message(chat_id, f"⚠️ Kuch gadbad hui: {e}", parse_mode="Markdown")

# ===== Start bot =====
print("🤖 Professional Bot chal raha hai...")
bot.polling(non_stop=True, interval=0)
