from telegram import Update
from telegram.ext import (
    ApplicationBuilder,
    CommandHandler,
    MessageHandler,
    filters,
    ContextTypes,
)

TOKEN = "8982827253:AAE_CxuuXcsf3_bFt3Ey0w9NgZXkxvM_MbE"

users = {}


async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id

    users[user_id] = {
        "step": "klik",
        "klik": 0,
        "terminal": 0,
        "berilganlar": [],
        "naqd": 0,
        "oylik": 0,
    }

    await update.message.reply_text(
        "Klik summalarni yuboring\n\n"
        "Misol:\n12+15+26+14+11"
    )


def hisobla(text):
    sonlar = text.split("+")
    jami = sum(int(x) for x in sonlar)
    return jami


async def handle(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    text = update.message.text

    if user_id not in users:
        await update.message.reply_text("/start bosing")
        return

    user = users[user_id]

    # KLIK
    if user["step"] == "klik":
        try:
            jami = hisobla(text)

            user["klik"] = jami
            user["step"] = "terminal"

            await update.message.reply_text(
                f"✅ Klik jami: {jami}\n\n"
                "Endi Terminal summalarni yuboring"
            )

        except:
            await update.message.reply_text(
                "Noto‘g‘ri format\n\n"
                "Misol:\n12+15+26"
            )

        return

    # TERMINAL
    elif user["step"] == "terminal":
        try:
            jami = hisobla(text)

            user["terminal"] = jami
            user["step"] = "berilgan"

            await update.message.reply_text(
                f"✅ Terminal jami: {jami}\n\n"
                "Kimga pul berildi?\n\n"
                "Misol:\nNonga 150"
            )

        except:
            await update.message.reply_text(
                "Noto‘g‘ri format\n\n"
                "Misol:\n12+15+47"
            )

        return

    # BERILGAN PULLAR
    elif user["step"] == "berilgan":

        if text.lower() == "ok":
            user["step"] = "naqd"

            await update.message.reply_text(
                "Naqd pul qancha?"
            )

            return

        try:
            parts = text.rsplit(" ", 1)

            nom = parts[0]
            summa = int(parts[1])

            user["berilganlar"].append((nom, summa))

            await update.message.reply_text(
                "✅ Saqlandi\n"
                "Yana yozing yoki tugasa ok deb yozing"
            )

        except:
            await update.message.reply_text(
                "Misol:\nNonga 150"
            )

        return

    # NAQD
    elif user["step"] == "naqd":
        try:
            naqd = int(text)

            user["naqd"] = naqd
            user["step"] = "oylik"

            await update.message.reply_text(
                "Oylik qancha?"
            )

        except:
            await update.message.reply_text(
                "Faqat son yuboring"
            )

        return

    # OYLIK
    elif user["step"] == "oylik":
        try:
            oylik = int(text)

            user["oylik"] = oylik

            klik = user["klik"]
            terminal = user["terminal"]
            naqd = user["naqd"]

            berilgan_text = ""
            berilgan_jami = 0

            for nom, summa in user["berilganlar"]:
                berilgan_text += f"{nom} - {summa}\n"
                berilgan_jami += summa

            umumiy = klik + terminal + naqd + berilgan_jami

            qolgan = umumiy - oylik

            result = (
                f"📊 HISOBOT\n\n"
                f"💳 Klik: {klik}\n"
                f"🏧 Terminal: {terminal}\n\n"
                f"💸 Berilgan pullar:\n"
                f"{berilgan_text}\n"
                f"💵 Naqd: {naqd}\n"
                f"👨‍💼 Oylik: {oylik}\n\n"
                f"📦 Berilgan jami: {berilgan_jami}\n"
                f"🧾 UMUMIY JAMI: {umumiy}\n\n"
                f"✅ QOLGAN PUL: {qolgan}"
            )

            await update.message.reply_text(result)

            users[user_id]["step"] = "done"

        except:
            await update.message.reply_text(
                "Faqat son yuboring"
            )


app = ApplicationBuilder().token(TOKEN).build()

app.add_handler(CommandHandler("start", start))
app.add_handler(MessageHandler(filters.TEXT, handle))

print("Bot ishladi...")

app.run_polling()
