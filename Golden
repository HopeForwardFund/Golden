import logging
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup, LabeledPrice
from telegram.ext import Application, CommandHandler, CallbackQueryHandler, MessageHandler, PreCheckoutQueryHandler, filters, ContextTypes
 
# =============================================
# ВСТАВЬТЕ СЮДА ВАШ ТОКЕН:
TOKEN = "8691958046:AAEM8xo4JvLgdg-OBu_W3kUo3swSfKCZsCs"  # токен от @BotFather
ADMIN_ID = 8596575829  # ваш Telegram ID
# =============================================
 
logging.basicConfig(level=logging.INFO)
 
# Хранилище контента
content = {
    1: {"photos": [], "videos": []},
    2: {"photos": [], "videos": []},
    3: {"photos": [], "videos": []},
    4: {"photos": [], "videos": []},
}
 
# Купленные пакеты {user_id: [1, 2, 3]}
purchased = {}
 
PACKAGES = {
    1: {"name_en": "Bronze", "name_ru": "Бронза", "photos": 10, "videos": 3, "stars": 250},
    2: {"name_en": "Silver", "name_ru": "Серебро", "photos": 15, "videos": 5, "stars": 500},
    3: {"name_en": "Gold",   "name_ru": "Золото",  "photos": 25, "videos": 10, "stars": 1000},
    4: {"name_en": "Platinum", "name_ru": "Платина", "photos": 50, "videos": 20, "stars": 1500},
}
 
TEXTS = {
    "en": {
        "welcome": """👣 *Welcome to GoldenFoot!*
 
💎 Exclusive foot content for true connoisseurs.
 
Choose your package and unlock premium content! 🔥""",
        "packages": """💎 *Choose your package:*
 
🥉 *Bronze — 250 ⭐*
• 10 photos + 3 videos
 
🥈 *Silver — 500 ⭐*
• 15 photos + 5 videos
 
🥇 *Gold — 1000 ⭐*
• 25 photos + 10 videos
 
💎 *Platinum — 1500 ⭐*
• 50 photos + 20 videos""",
        "buying": "💫 Processing payment...",
        "activated": "✅ *Package activated!*\n\nUse /content{} to get your content!",
        "no_content": "⏳ Content is being prepared. Check back soon!",
        "already": "✅ You already have this package! Use /content{}",
        "select_lang": "🌍 Choose language / Выберите язык",
    },
    "ru": {
        "welcome": """👣 *Добро пожаловать в GoldenFoot!*
 
💎 Эксклюзивный контент для настоящих ценителей.
 
Выберите пакет и получите доступ к премиум контенту! 🔥""",
        "packages": """💎 *Выберите пакет:*
 
🥉 *Бронза — 250 ⭐*
• 10 фото + 3 видео
 
🥈 *Серебро — 500 ⭐*
• 15 фото + 5 видео
 
🥇 *Золото — 1000 ⭐*
• 25 фото + 10 видео
 
💎 *Платина — 1500 ⭐*
• 50 фото + 20 видео""",
        "buying": "💫 Обрабатываю платёж...",
        "activated": "✅ *Пакет активирован!*\n\nИспользуйте /content{} чтобы получить контент!",
        "no_content": "⏳ Контент готовится. Заходите позже!",
        "already": "✅ У вас уже есть этот пакет! Используйте /content{}",
        "select_lang": "🌍 Choose language / Выберите язык",
    }
}
 
user_langs = {}
admin_states = {}
 
def get_lang(user_id):
    return user_langs.get(user_id, "en")
 
def get_purchased(user_id):
    return purchased.get(user_id, [])
 
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    keyboard = [
        [InlineKeyboardButton("🇬🇧 English", callback_data="lang_en"),
         InlineKeyboardButton("🇷🇺 Русский", callback_data="lang_ru")]
    ]
    await update.message.reply_text(
        TEXTS["en"]["select_lang"],
        reply_markup=InlineKeyboardMarkup(keyboard)
    )
 
async def show_packages(update, lang, user_id):
    pkgs = get_purchased(user_id)
    keyboard = []
    for pkg_id, pkg in PACKAGES.items():
        name = pkg[f"name_{lang}"]
        stars = pkg["stars"]
        owned = "✅ " if pkg_id in pkgs else ""
        keyboard.append([InlineKeyboardButton(
            f"{owned}{name} — {stars} ⭐",
            callback_data=f"buy_{pkg_id}"
        )])
    keyboard.append([InlineKeyboardButton("📦 My Content", callback_data="my_content")])
 
    if update.callback_query:
        await update.callback_query.message.reply_text(
            TEXTS[lang]["packages"],
            parse_mode="Markdown",
            reply_markup=InlineKeyboardMarkup(keyboard)
        )
    else:
        await update.message.reply_text(
            TEXTS[lang]["packages"],
            parse_mode="Markdown",
            reply_markup=InlineKeyboardMarkup(keyboard)
        )
 
async def button(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()
    user_id = query.from_user.id
    data = query.data
 
    if data.startswith("lang_"):
        lang = data.split("_")[1]
        user_langs[user_id] = lang
        await query.message.reply_text(
            TEXTS[lang]["welcome"],
            parse_mode="Markdown"
        )
        await show_packages(update, lang, user_id)
 
    elif data.startswith("buy_"):
        pkg_id = int(data.split("_")[1])
        lang = get_lang(user_id)
        pkgs = get_purchased(user_id)
 
        if pkg_id in pkgs:
            await query.message.reply_text(
                TEXTS[lang]["already"].format(pkg_id),
                parse_mode="Markdown"
            )
            return
 
        pkg = PACKAGES[pkg_id]
        name = pkg[f"name_{lang}"]
        await context.bot.send_invoice(
            chat_id=user_id,
            title=f"GoldenFoot — {name}",
            description=f"{pkg['photos']} photos + {pkg['videos']} videos",
            payload=f"pkg_{pkg_id}",
            provider_token="",
            currency="XTR",
            prices=[LabeledPrice(label=name, amount=pkg["stars"])]
        )
 
    elif data == "my_content":
        lang = get_lang(user_id)
        pkgs = get_purchased(user_id)
        if not pkgs:
            await query.message.reply_text("❌ You have no packages yet. Buy one first!")
            return
        text = "📦 *Your packages:*\n\n"
        for pkg_id in pkgs:
            pkg = PACKAGES[pkg_id]
            name = pkg[f"name_{lang}"]
            text += f"• {name} → /content{pkg_id}\n"
        await query.message.reply_text(text, parse_mode="Markdown")
 
async def precheckout(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.pre_checkout_query.answer(ok=True)
 
async def successful_payment(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    lang = get_lang(user_id)
    payload = update.message.successful_payment.invoice_payload
    pkg_id = int(payload.split("_")[1])
 
    if user_id not in purchased:
        purchased[user_id] = []
    if pkg_id not in purchased[user_id]:
        purchased[user_id].append(pkg_id)
 
    await update.message.reply_text(
        TEXTS[lang]["activated"].format(pkg_id),
        parse_mode="Markdown"
    )
 
async def send_content(update: Update, context: ContextTypes.DEFAULT_TYPE, pkg_id: int):
    user_id = update.effective_user.id
    lang = get_lang(user_id)
    pkgs = get_purchased(user_id)
 
    if pkg_id not in pkgs:
        keyboard = [[InlineKeyboardButton("💎 Buy Package", callback_data=f"buy_{pkg_id}")]]
        await update.message.reply_text(
            "❌ You don't have this package!\n\nBuy it to get access:",
            reply_markup=InlineKeyboardMarkup(keyboard)
        )
        return
 
    photos = content[pkg_id]["photos"]
    videos = content[pkg_id]["videos"]
 
    if not photos and not videos:
        await update.message.reply_text(TEXTS[lang]["no_content"])
        return
 
    await update.message.reply_text(f"📦 Sending your content... ({len(photos)} photos, {len(videos)} videos)")
 
    for photo_id in photos:
        try:
            await update.message.reply_photo(photo=photo_id)
        except:
            pass
 
    for video_id in videos:
        try:
            await update.message.reply_video(video=video_id)
        except:
            pass
 
async def content1(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await send_content(update, context, 1)
 
async def content2(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await send_content(update, context, 2)
 
async def content3(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await send_content(update, context, 3)
 
async def content4(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await send_content(update, context, 4)
 
# ===== ADMIN COMMANDS =====
 
async def add1(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await handle_add(update, context, 1)
 
async def add2(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await handle_add(update, context, 2)
 
async def add3(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await handle_add(update, context, 3)
 
async def add4(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await handle_add(update, context, 4)
 
async def handle_add(update: Update, context: ContextTypes.DEFAULT_TYPE, pkg_id: int):
    user_id = update.effective_user.id
    if user_id != ADMIN_ID:
        await update.message.reply_text("❌ No access!")
        return
    admin_states[user_id] = pkg_id
    pkg = PACKAGES[pkg_id]
    await update.message.reply_text(
        f"📤 Send photo or video for Package {pkg_id} ({pkg['name_en']})\n\n"
        f"Current: {len(content[pkg_id]['photos'])} photos, {len(content[pkg_id]['videos'])} videos\n\n"
        f"Send /cancel to stop adding."
    )
 
async def cancel(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    if user_id in admin_states:
        del admin_states[user_id]
    await update.message.reply_text("✅ Done adding content.")
 
async def stats(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    if user_id != ADMIN_ID:
        return
    total_users = len(purchased)
    total_sales = sum(len(pkgs) for pkgs in purchased.values())
    text = f"📊 *Stats:*\n\n"
    text += f"👥 Buyers: {total_users}\n"
    text += f"💰 Total sales: {total_sales}\n\n"
    for pkg_id, pkg in PACKAGES.items():
        count = sum(1 for pkgs in purchased.values() if pkg_id in pkgs)
        stars = count * pkg["stars"]
        text += f"• {pkg['name_en']}: {count} sales = {stars}⭐\n"
    text += f"\n📦 *Content:*\n"
    for pkg_id in range(1, 5):
        text += f"• Package {pkg_id}: {len(content[pkg_id]['photos'])} photos, {len(content[pkg_id]['videos'])} videos\n"
    await update.message.reply_text(text, parse_mode="Markdown")
 
async def handle_media(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    if user_id != ADMIN_ID or user_id not in admin_states:
        return
 
    pkg_id = admin_states[user_id]
 
    if update.message.photo:
        file_id = update.message.photo[-1].file_id
        content[pkg_id]["photos"].append(file_id)
        count = len(content[pkg_id]["photos"])
        await update.message.reply_text(f"✅ Photo added to Package {pkg_id}! Total photos: {count}")
 
    elif update.message.video:
        file_id = update.message.video.file_id
        content[pkg_id]["videos"].append(file_id)
        count = len(content[pkg_id]["videos"])
        await update.message.reply_text(f"✅ Video added to Package {pkg_id}! Total videos: {count}")
 
async def packages(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    lang = get_lang(user_id)
    await show_packages(update, lang, user_id)
 
def main():
    if TOKEN == "ВСТАВЬТЕ_ТОКЕН_БОТА_СЮДА":
        print("❌ Вставьте токен бота!")
        return
 
    app = Application.builder().token(TOKEN).build()
 
    # User commands
    app.add_handler(CommandHandler("start", start))
    app.add_handler(CommandHandler("packages", packages))
    app.add_handler(CommandHandler("content1", content1))
    app.add_handler(CommandHandler("content2", content2))
    app.add_handler(CommandHandler("content3", content3))
    app.add_handler(CommandHandler("content4", content4))
 
    # Admin commands
    app.add_handler(CommandHandler("add1", add1))
    app.add_handler(CommandHandler("add2", add2))
    app.add_handler(CommandHandler("add3", add3))
    app.add_handler(CommandHandler("add4", add4))
    app.add_handler(CommandHandler("cancel", cancel))
    app.add_handler(CommandHandler("stats", stats))
 
    # Handlers
    app.add_handler(CallbackQueryHandler(button))
    app.add_handler(PreCheckoutQueryHandler(precheckout))
    app.add_handler(MessageHandler(filters.SUCCESSFUL_PAYMENT, successful_payment))
    app.add_handler(MessageHandler(filters.PHOTO | filters.VIDEO, handle_media))
 
    print("✅ GoldenFoot Bot запущен!")
    app.run_polling()
 
if __name__ == "__main__":
    main()
