# Bot setup so the barcode gets a reply

The barcode scanner sends data with `tg.sendData()`. For the **bot to receive it and reply**, two things must be correct.

## 1. Open the Web App from a **Reply Keyboard** button (not inline)

`sendData()` only works when the user opens the mini app from a **custom keyboard button** (Reply Keyboard), not from an **inline keyboard** or **menu button**.

Example (Python, aiogram 3):

```python
from aiogram.types import WebAppInfo, ReplyKeyboardMarkup, KeyboardButton

@router.message(Command("start"))
async def cmd_start(message: Message):
    keyboard = ReplyKeyboardMarkup(
        keyboard=[
            [KeyboardButton(text="📷 مسح الباركود", web_app=WebAppInfo(url="https://YOUR-SITE.com/index.html"))]
        ],
        resize_keyboard=True,
    )
    await message.answer("اضغط الزر لفتح الماسح:", reply_markup=keyboard)
```

## 2. Handle `web_app_data` in the bot and reply

When the user sends the barcode from the Web App, the bot gets a **message** with `message.web_app_data`. You must handle it and call `message.answer()` so the user sees a reply.

**Python (aiogram 3):**

```python
import json
from aiogram import F
from aiogram.types import Message

@router.message(F.web_app_data)
async def handle_barcode(message: Message):
    data = message.web_app_data.data  # string from tg.sendData()
    try:
        payload = json.loads(data)
        barcode = payload.get("barcode", data)
    except Exception:
        barcode = data
    await message.answer(f"✅ تم استلام الباركود: {barcode}")
```

**Node (Telegraf):**

```js
bot.on('message', async (ctx) => {
  if (!ctx.message.web_app_data) return;
  let barcode = ctx.message.web_app_data.data;
  try {
    const payload = JSON.parse(barcode);
    barcode = payload.barcode || barcode;
  } catch (_) {}
  await ctx.reply(`✅ تم استلام الباركود: ${barcode}`);
});
```

After adding this handler and opening the app from the Reply Keyboard button, clicking "إرسال هذا الباركود" will send the barcode and the bot will reply in the chat.
