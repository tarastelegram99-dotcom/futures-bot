import requests
import random
import asyncio
from telegram import Bot

TOKEN = "8598664810:AAFsWXBtf7OIM01c8wJ16XsQHZ023U0nQQ"   # твой токен
MY_ID = 6096618599                                        # твой ID — уже правильный
IMAGE = "signal.jpg"

def get_price(symbol):
    try:
        r = requests.get(f"https://fapi.binance.com/fapi/v1/ticker/price?symbol={symbol}USDT")
        return round(float(r.json()["price"]), 2 if symbol == "SOL" else 1)
    except:
        return 108000 if symbol == "BTC" else 4700 if symbol == "ETH" else 180

async def send_signal():
    bot = Bot(TOKEN)
    coin = random.choice(["BTC", "ETH", "SOL"])
    price = get_price(coin)
    direction = random.choice(["LONG", "SHORT"])
    is_long = direction == "LONG"

    entry = round(price * random.uniform(0.998, 1.002), 2 if coin == "SOL" else 1)
    tp    = round(entry * (1.045 if is_long else 0.955), 2 if coin == "SOL" else 1)
    sl    = round(entry * (0.985 if is_long else 1.015), 2 if coin == "SOL" else 1)

    coin_emo = "Bitcoin" if coin == "BTC" else "Ethereum" if coin == "ETH" else "Sun"
    dir_emo  = "LONG" if is_long else "SHORT"

    caption = (
        f"Сигнал на {coin_emo} {coin}\n\n"
        f"Направление {direction} {dir_emo}\n"
        f"Точка входа `{entry:,}$`\n"
        f"Take Profit `{tp:,}$`\n"
        f"Stop Loss `{sl:,}$`\n\n"
        "Плечо 5–20× • Риск 1–2% от депозита"
    )

    with open(IMAGE, "rb") as photo:
        await bot.send_photo(chat_id=MY_ID, photo=photo, caption=caption, parse_mode="Markdown")

async def main():
    print("Бот живёт 24/7 — сигналы каждые 5 минут")
    while True:
        try:
            await send_signal()
        except Exception as e:
            print("Ошибка, но продолжаем:", e)
        await asyncio.sleep(300)

if __name__ == "__main__":
    asyncio.run(main())
