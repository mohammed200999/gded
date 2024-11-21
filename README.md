import telebot

TOKEN = "7220674073:AAEGpK7jTd3xCTu1fni1-UAcPiHT-v5Yw7I"  # ضع التوكن الخاص ببوتك هنا
bot = telebot.TeleBot(TOKEN)

ADMIN_ID = 5643717731  # ضع معرفك هنا
# قواعد بيانات النقاط والقنوات
user_points = {}  # تخزين النقاط لكل القنواتت
user_points[ADMIN_ID] = 10_000_000  # إضافة 10 مليون نقطة للمالك
channels = []  # قائمة القنوات

# أمر /start: ترحيب وتوضيح النقاط
@bot.message_handler(commands=['start'])
def start(message):
    user_id = message.chat.id
    if user_id not in user_points:
        user_points[user_id] = 0  # المستخدم الجديد يبدأ بـ 0 نقطة
    bot.send_message(
        user_id,
        "مرحبًا بك في بوت التمويل! 🌟\n"
        "• لجمع النقاط: اضغط /collect\n"
        "• لمعرفة نقاطك: اضغط /points\n"
        "• لتمويل قناتك: اضغط /fund\n"
        "• لإضافة قناة جديدة: اضغط /add_channel\n\n"
        "ابدأ الآن واكسب النقاط!"
    )

# أمر /points: عرض نقاط المستخدم
@bot.message_handler(commands=['points'])
def points(message):
    user_id = message.chat.id
    points = user_points.get(user_id, 0)
    bot.send_message(user_id, f"✨ نقاطك الحالية: {points} نقطة.")

# أمر /collect: جمع النقاط من الاشتراك بالقنوات
@bot.message_handler(commands=['collect'])
def collect(message):
    user_id = message.chat.id
    if not channels:
        bot.send_message(user_id, "🚫 لا توجد قنوات لجمع النقاط حاليًا.")
        return

    channel = channels[0]  # أول قناة من القائمة
    bot.send_message(
        user_id,
        f"🔗 اشترك بالقناة التالية لجمع 20 نقطة:\n{channel}\n\n"
        "بعد الاشتراك، اضغط /done."
    )

# أمر /done: التأكد من الاشتراك وإضافة النقاط
@bot.message_handler(commands=['done'])
def done(message):
    user_id = message.chat.id
    if user_id not in user_points:
        user_points[user_id] = 0

    user_points[user_id] += 20  # إضافة 20 نقطة
    bot.send_message(user_id, "✅ تم إضافة 20 نقطة إلى حسابك! 🎉")

# أمر /add_channel: إضافة قناة جديدة
@bot.message_handler(commands=['add_channel'])
def add_channel(message):
    user_id = message.chat.id
    bot.send_message(user_id, "📥 أرسل رابط القناة التي تريد إضافتها.")
    bot.register_next_step_handler(message, save_channel)

def save_channel(message):
    channel = message.text
    channels.append(channel)
    bot.send_message(message.chat.id, f"✅ تم إضافة القناة: {channel}")

# أمر /fund: تمويل قناة بالنقاط
@bot.message_handler(commands=['fund'])
def fund(message):
    user_id = message.chat.id
    points = user_points.get(user_id, 0)
    if points < 20:
        bot.send_message(user_id, "🚫 لا تمتلك نقاطًا كافية للتمويل (الحد الأدنى: 20 نقطة).")
        return

    bot.send_message(user_id, "📥 أرسل رابط القناة التي تريد تمويلها.")
    bot.register_next_step_handler(message, process_funding)

def process_funding(message):
    user_id = message.chat.id
    channel = message.text
    if channel not in channels:
        bot.send_message(user_id, "🚫 القناة غير موجودة في النظام.")
        return

    user_points[user_id] -= 20  # خصم 20 نقطة
    bot.send_message(
        user_id,
        f"✅ تم تمويل القناة: {channel} بـ 20 نقطة.\n"
        f"✨ نقاطك المتبقية: {user_points[user_id]} نقطة."
    )
