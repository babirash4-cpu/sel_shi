# Telegram selfbot betting — Railway deployment

متغیرهای محیطی که در پنل Railway (Variables tab) باید تنظیم کنی:

## اجباری

| متغیر | مقدار |
|---|---|
| `BOT_TOKEN` | توکن ربات اصلی از @BotFather |
| `TELEGRAM_API_ID` | api_id از my.telegram.org |
| `TELEGRAM_API_HASH` | api_hash از my.telegram.org |
| `OWNER_ID` | numeric chat-id اکانت مدیریت |
| `SESSION_ENCRYPTION_KEY` | کلید Fernet برای رمزنگاری سشن‌ها — `python3 -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"` |

## اختیاری (پیش‌فرض‌هاشون کار می‌کنه)

`BOT_DATA_DIR`, `BOT_SESSIONS_DIR`, `BOT_TIMEZONE` (پیش‌فرض `Asia/Tehran`), `BETTING_*`, `MAX_IN_MEMORY_MEDIA_MB`, `SELF_*`, `HELPER_*`.

## نکات Railway

- **Volume**: یک Volume بساز و mount path رو `/data` بذار. دیتابیس‌ها و سشن‌ها اونجا نوشته می‌شن (`RAILWAY_VOLUME_MOUNT_PATH=/data` خودکار ست می‌شه) و با restart از بین نمی‌رن. بدون volume، همه چیز روی filesystem موقت نوشته می‌شه و با هر deploy پاک می‌شه.
- **توکن داخل دیتابیس**: توکن ربات هلپر و نام کاربری‌اش داخل دیتابیس (`helper_config`) ذخیره می‌شن نه env. بعد از اولین start، توکن هلپر رو از طریق پنل ادمین ربات اصلی ثبت کن (یا مستقیماً داخل users.db).
- **API ID/Hash سلف‌ها**: هر سلف‌باتی که با Telethon اجرا می‌شه از api_id/hash همان اکانت استفاده می‌کنه که موقع add کردن کاربر وارد می‌شه (داخل دیتابیس ذخیره می‌شه). API credentials اصلی صرفاً برای ربات اصلی PTB است.
- **Health check**: ربات از long-polling استفاده می‌کنه و وب‌سرور ندارد. Railway ممکنه در مورد نبود PORT هشدار بده؛ مشکلی نیست، deploy همچنان کار می‌کنه. اگر Railway روی health check پایدار اصرار کرد، `PORT` رو روی یک عدد دلخواه (مثل `8080`) تنظیم کن — کد به آن گوش نمی‌دهد ولی قانون پلتفرم برآورده می‌شود.
- **ساختار**: `main_bot.py` در startup هلپر و سلف‌ها رو spawn می‌کنه. فقط یک سرویس بساز؛ همه چی از همون یک process اجرا می‌شه.
- **ایمنی restart**: `restartPolicyType: ON_FAILURE` تنظیم شده.
- این ریپو حاوی منطق شرط‌بندی/قمار هست؛ قوانین GitHub و Telegram درباره محتوا رو رعایت کن.
