import requests
import time
from datetime import datetime

# ===== আপনার তথ্য =====
BOT_TOKEN = "8492847304:AAFbN-gvCc8Ln2q0AaMq5rxyCK-0nzWDWwg"
CHANNEL_ID = "-1004398538737"
CHECK_INTERVAL = 300  # ৫ মিনিট = ৩০০ সেকেন্ড

# ===== Gold Price নেওয়া =====
def get_gold_price():
    try:
        url = "https://query1.finance.yahoo.com/v8/finance/chart/GC=F"
        headers = {"User-Agent": "Mozilla/5.0"}
        response = requests.get(url, headers=headers, timeout=10)
        data = response.json()
        price = data["chart"]["result"][0]["meta"]["regularMarketPrice"]
        return round(price, 2)
    except Exception as e:
        print(f"Price fetch error: {e}")
        return None

# ===== Telegram Message পাঠানো =====
def send_telegram(message):
    url = f"https://api.telegram.org/bot{BOT_TOKEN}/sendMessage"
    payload = {
        "chat_id": CHANNEL_ID,
        "text": message,
        "parse_mode": "HTML"
    }
    try:
        requests.post(url, json=payload, timeout=10)
        print("Message sent!")
    except Exception as e:
        print(f"Send error: {e}")

# ===== Signal তৈরি করা =====
def analyze_signal(current_price, previous_price):
    if previous_price is None:
        return None
    
    change = current_price - previous_price
    change_pct = (change / previous_price) * 100

    now = datetime.utcnow().strftime("%H:%M UTC")

    if change_pct >= 0.15:
        signal = "🟢 BUY SIGNAL"
        entry = round(current_price, 2)
        tp1 = round(current_price + 5, 2)
        tp2 = round(current_price + 10, 2)
        sl = round(current_price - 8, 2)
        emoji = "📈"
    elif change_pct <= -0.15:
        signal = "🔴 SELL SIGNAL"
        entry = round(current_price, 2)
        tp1 = round(current_price - 5, 2)
        tp2 = round(current_price - 10, 2)
        sl = round(current_price + 8, 2)
        emoji = "📉"
    else:
        return None  # কোনো বড় মুভমেন্ট নেই

    message = f"""
{emoji} <b>GOLD (XAUUSD) {signal}</b> {emoji}

💰 <b>Current Price:</b> ${current_price}
🎯 <b>Entry:</b> ${entry}
✅ <b>TP1:</b> ${tp1}
✅ <b>TP2:</b> ${tp2}
❌ <b>SL:</b> ${sl}

📊 <b>Change:</b> {change_pct:+.2f}%
🕐 <b>Time:</b> {now}

⚠️ <i>Risk management মেনে trade করুন!</i>
📌 @goldrana_signals
"""
    return message

# ===== Main Loop =====
def main():
    print("🤖 Gold Signal Bot চালু হয়েছে!")
    send_telegram("🤖 <b>Gold Signal Bot চালু হয়েছে!</b>\n\nপ্রতি ৫ মিনিটে XAUUSD সিগন্যাল পাবেন। 📊")
    
    previous_price = None

    while True:
        current_price = get_gold_price()
        
        if current_price:
            print(f"Gold Price: ${current_price}")
            signal_msg = analyze_signal(current_price, previous_price)
            
            if signal_msg:
                send_telegram(signal_msg)
            
            previous_price = current_price
        
        time.sleep(CHECK_INTERVAL)

if __name__ == "__main__":
    main()
