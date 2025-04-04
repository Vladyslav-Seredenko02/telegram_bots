# telegram_bots
from telegram import Update, ReplyKeyboardMarkup
from telegram.ext import ApplicationBuilder, CommandHandler, MessageHandler, ContextTypes, filters

# Переменная для хранения выбранного языка (по умолчанию русский)
user_languages = {}

# Кнопки меню
menu_keyboard_ru = [
    ["🛒 Купить", "💰 Продать"],
    ["🆘 Техподдержка", "ℹ Инфо"],
    ["🌐 Сменить язык"]  # Кнопка для смены языка
]

menu_keyboard_en = [
    ["🛒 Buy", "💰 Sell"],
    ["🆘 Support", "ℹ Info"],
    ["🌐 Change language"]  # Button to change language
]

# Клавиатуры для каждого языка
reply_markup_ru = ReplyKeyboardMarkup(menu_keyboard_ru, resize_keyboard=True)
reply_markup_en = ReplyKeyboardMarkup(menu_keyboard_en, resize_keyboard=True)

# Тексты для разных языков
texts = {
    'ru': {
        'start': "Добро пожаловать в магазин! Выберите действие:",
        'buy': "Для того чтобы купить товар, нажмите на [Купить товар](https://example.com)",
        'sell': "Чтобы продать товар, отправьте описание и фото товара. Мы свяжемся с вами для дальнейших шагов.",
        'info': "Информации нету, разбирайтесь сами.",
        'support': "Свяжитесь с нами: @vlad_seredenko, @lusshiffer",
        'language': "Вы выбрали русский язык.",
        'change_language': "Чтобы сменить язык, нажмите кнопку '🌐 Сменить язык'."
    },
    'en': {
        'start': "Welcome to the store! Choose an action:",
        'buy': "To buy the product, click on [Buy Product](https://example.com)",
        'sell': "To sell a product, send the description and photo. We will contact you for further steps.",
        'info': "No information, figure it out yourself.",
        'support': "Contact us: @vlad_seredenko, @lusshiffer",
        'language': "You have selected the English language.",
        'change_language': "To change the language, press the '🌐 Change language' button."
    }
}

# Команда /start
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.message.from_user.id
    language = user_languages.get(user_id, 'ru')  # Если языка нет в словаре, по умолчанию русский
    if language == 'ru':
        await update.message.reply_text(texts[language]['start'], reply_markup=reply_markup_ru)
    else:
        await update.message.reply_text(texts[language]['start'], reply_markup=reply_markup_en)

# Команда для кнопки "Купить"
async def buy(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.message.from_user.id
    language = user_languages.get(user_id, 'ru')
    if language == 'ru':
        await update.message.reply_text(texts[language]['buy'], reply_markup=reply_markup_ru, parse_mode="Markdown")
    else:
        await update.message.reply_text(texts[language]['buy'], reply_markup=reply_markup_en, parse_mode="Markdown")

# Команда для кнопки "Продать"
async def sell(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.message.from_user.id
    language = user_languages.get(user_id, 'ru')
    if language == 'ru':
        await update.message.reply_text(texts[language]['sell'], reply_markup=reply_markup_ru)
    else:
        await update.message.reply_text(texts[language]['sell'], reply_markup=reply_markup_en)

# Команда для кнопки "Инфо"
async def info(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.message.from_user.id
    language = user_languages.get(user_id, 'ru')
    if language == 'ru':
        await update.message.reply_text(texts[language]['info'], reply_markup=reply_markup_ru)
    else:
        await update.message.reply_text(texts[language]['info'], reply_markup=reply_markup_en)

# Команда для кнопки "Техподдержка"
async def support(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.message.from_user.id
    language = user_languages.get(user_id, 'ru')
    if language == 'ru':
        await update.message.reply_text(texts[language]['support'], reply_markup=reply_markup_ru)
    else:
        await update.message.reply_text(texts[language]['support'], reply_markup=reply_markup_en)

# Команда для смены языка
async def change_language(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.message.from_user.id
    current_language = user_languages.get(user_id, 'ru')
    
    # Переключение языка
    new_language = 'en' if current_language == 'ru' else 'ru'
    user_languages[user_id] = new_language
    
    # Отправка сообщения о смене языка
    if new_language == 'ru':
        await update.message.reply_text(texts[new_language]['language'], reply_markup=reply_markup_ru)
    else:
        await update.message.reply_text(texts[new_language]['language'], reply_markup=reply_markup_en)

# Обработка других сообщений
async def handle_message(update: Update, context: ContextTypes.DEFAULT_TYPE):
    text = update.message.text
    user_id = update.message.from_user.id
    language = user_languages.get(user_id, 'ru')

    if text == "🛒 Купить" or text == "🛒 Buy":
        await buy(update, context)
    elif text == "💰 Продать" or text == "💰 Sell":
        await sell(update, context)
    elif text == "🆘 Техподдержка" or text == "🆘 Support":
        await support(update, context)
    elif text == "ℹ Инфо" or text == "ℹ Info":
        await info(update, context)
    elif text == "🌐 Сменить язык" or text == "🌐 Change language":
        await change_language(update, context)
    else:
        if language == 'ru':
            await update.message.reply_text("Выберите действие из меню!", reply_markup=reply_markup_ru)
        else:
            await update.message.reply_text("Please choose an action from the menu!", reply_markup=reply_markup_en)

def main():
    TOKEN = "7732429245:AAH3YxP1MhYGdy91ZCZ5HFUuihSpWGznvPw"
    
    app = ApplicationBuilder().token(TOKEN).build()

    app.add_handler(CommandHandler("start", start))
    app.add_handler(CommandHandler("language", change_language))  # Обработчик команды для смены языка
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_message))

    print("Бот запущен...")
    app.run_polling()

if __name__ == '__main__':
    main()
