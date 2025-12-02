import requests
import random
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import ApplicationBuilder, CommandHandler, CallbackQueryHandler, ContextTypes

TOKEN = "8598664810:AAFsWXBtf7OIM01c8wJ16XsQHZ023U0nQQ"
MANAGER = "@David01w"
IMAGE = "signal.jpg"

def get_price(symbol):
    try:
        r = requests.get(f"https://fapi.binance.com/fapi/v1/ticker/price?symbol={symbol}USDT", timeout=10)
        return round(float(r.json()["price"]), 2 if symbol == "SOL" else 1)
    except:
        return 108000 if symbol == "BTC" else 4700 if symbol == "ETH" else 180

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if context.user_data.get("done"):
        await update.message.reply_text(f"Ты уже получил сигнал сегодня\nОбращайся к аналитику {MANAGER}")
        return
    kb = [
        [InlineKeyboardButton("Русский", callback_data="lang_ru")],
        [InlineKeyboardButton("English", callback_data="lang_en")]
    ]
    await update.message.reply_text("Привет!\nВыберите язык:", reply_markup=InlineKeyboardMarkup(kb))

async def button(update: Update, context: ContextTypes.DEFAULT_TYPE):
    q = update.callback_query
    await q.answer()
    data = q.data

    if data in ["lang_ru", "lang_en"]:
        context.user_data["lang"] = "ru" if data == "lang_ru" else "en"
        kb = [
            [InlineKeyboardButton("BTC", callback_data="coin_BTC")],
            [InlineKeyboardButton("ETH", callback_data="coin_ETH")],
            [InlineKeyboardButton("SOL", callback_data="coin_SOL")]
        ]
        await q.edit_message_text("Выбери монету", reply_markup=InlineKeyboardMarkup(kb))
        return

    if data.startswith("coin_"):
        if context.user_data.get("done"):
            await q.edit_message_text("Ты уже получил сигнал сегодня")
            return

        coin = data.split("_")[1]
        context.user_data["done"] = True
        price = get_price(coin)
        direction = random.choice(["LONG", "SHORT"])
        is_long = direction == "LONG"

        entry = round(price * random.uniform(0.998, 1.002), 2 if coin == "SOL" else 1)
        tp = round(entry * (1.045 if is_long else 0.955), 2 if coin == "SOL" else 1)
        sl = round(entry * (0.985 if is_long else 1.015), 2 if coin == "SOL" else 1)

        coin_emo = "Bitcoin" if coin == "BTC" else "Ethereum" if coin == "ETH" else "Sun"

        caption = (
            f"Сигнал на {coin_emo} {coin}\n\n"
            f"Направление {direction} {'LONG' if is_long else 'SHORT'}\n"
            f"Точка входа `{entry:,}$`\n"
            f"Take Profit `{tp:,}$`\n"
            f"Stop Loss `{sl:,}$`\n\n"
            "Плечо 5–20× • Риск 1–2%"
        )

        kb = [[InlineKeyboardButton("Дальше", callback_data="manager")]]

        with open(IMAGE, "rb") as photo:
            await q.message.reply_photo(photo=photo, caption=caption, parse_mode="Markdown", reply_markup=InlineKeyboardMarkup(kb))
        return

    if data == "manager":
        await q.edit_message_caption(
            caption=q.message.caption + f"\n\nДля доступа к профессиональным сигналам — свяжитесь с аналитиком {MANAGER}",
            parse_mode="Markdown"
        )
        await q.edit_message_reply_markup(reply_markup=None)

app = ApplicationBuilder().token(TOKEN).concurrent_updates=True.build()
app.add_handler(CommandHandler("start", start))
app.add_handler(CallbackQueryHandler(button))

print("Бот запущен 24/7 — всё работает!")
app.run_polling()
