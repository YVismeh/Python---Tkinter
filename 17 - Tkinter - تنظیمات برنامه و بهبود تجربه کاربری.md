---
title: "Tkinter - جلسه ۱۷ - تنظیمات برنامه و بهبود تجربه کاربری"
aliases:
  - Tkinter Session 17
  - Application Settings and UX
tags:
  - Python
  - Tkinter
  - SQLite
  - JSON
  - آموزش
level: intermediate
status: ready
---

# جلسه ۱۷ — تنظیمات برنامه و بهبود تجربه کاربری

## هدف جلسه

در جلسات گذشته، رابط گرافیکی مدیریت دانشجو را ساختیم، داده‌ها را در SQLite ذخیره کردیم و برای اعتبارسنجی و مدیریت خطاها ساختار مناسب‌تری در نظر گرفتیم. در این جلسه روی تنظیمات برنامه و تجربه کاربری کار می‌کنیم.

در پایان جلسه، دانشجو می‌تواند:

- تنظیمات ساده برنامه را در یک فایل JSON ذخیره و بارگذاری کند.
- هنگام اجرای بعدی، تنظیمات قبلی را بازیابی کند.
- برای تنظیمات از مقادیر پیش‌فرض استفاده کند.
- خطاهای خواندن و نوشتن فایل تنظیمات را مدیریت کند.
- اندازه پنجره و وضعیت برخی گزینه‌ها را به شکل قابل نگهداری ذخیره کند.
- پیام‌های مناسب و بازخورد واضح‌تری به کاربر ارائه دهد.

## ۱. چرا برنامه به تنظیمات نیاز دارد؟

برنامه‌ای که در هر بار اجرا اندازه پنجره، گزینه‌ها و رفتار آن به حالت اولیه برمی‌گردد، ممکن است استفاده از آن دشوار باشد. تنظیمات می‌توانند مواردی مانند این‌ها را نگهداری کنند:

- اندازه و موقعیت پنجره
- حالت نمایش یا گزینه‌های انتخابی
- تعداد رکوردهای نمایش‌داده‌شده در هر صفحه
- ترجیحات ظاهری برنامه

اطلاعات اصلی دانشجو در SQLite ذخیره می‌شود، اما تنظیمات ساده برنامه را می‌توان در فایل JSON نگهداری کرد. این دو نوع داده را با هم اشتباه نگیرید: پایگاه داده برای داده‌های اصلی برنامه است و JSON در این پروژه برای ترجیحات سبک کاربر استفاده می‌شود.

## ۲. ساختار فایل تنظیمات

فایلی به نام `settings.json` می‌تواند چنین محتوایی داشته باشد:

```json
{
  "window_geometry": "720x420",
  "show_tips": true
}
```

JSON فقط نوع‌های داده‌ای مشخصی مانند رشته، عدد، بولی، آرایه، شیء و `null` را پشتیبانی می‌کند. در پایتون برای کار با آن از ماژول استاندارد `json` استفاده می‌کنیم.

## ۳. بارگذاری تنظیمات با مقدارهای پیش‌فرض

ممکن است فایل تنظیمات هنوز ساخته نشده باشد یا محتوای آن خراب شده باشد. بنابراین برنامه باید بتواند بدون آن نیز اجرا شود.

```python
import json
from pathlib import Path

SETTINGS_FILE = Path("settings.json")

DEFAULT_SETTINGS = {
    "window_geometry": "720x420",
    "show_tips": True
}


def load_settings():
    settings = DEFAULT_SETTINGS.copy()

    if not SETTINGS_FILE.exists():
        return settings

    try:
        with SETTINGS_FILE.open("r", encoding="utf-8") as file:
            saved_settings = json.load(file)

        if isinstance(saved_settings, dict):
            settings.update(saved_settings)

    except (OSError, json.JSONDecodeError):
        return DEFAULT_SETTINGS.copy()

    return settings
```

در این تابع، مقدارهای پیش‌فرض ابتدا کپی می‌شوند و سپس تنظیمات ذخیره‌شده روی آن‌ها اعمال می‌شوند. بنابراین اگر نسخه جدید برنامه تنظیمی تازه اضافه کند، مقدار پیش‌فرض آن همچنان وجود خواهد داشت.

نکته: در برنامه کامل‌تر، بهتر است نوع و محدوده هر تنظیم نیز بررسی شود؛ چون یک فایل JSON می‌تواند معتبر باشد اما مقدار نامناسبی داشته باشد.

## ۴. ذخیره تنظیمات

```python
def save_settings(settings):
    try:
        with SETTINGS_FILE.open("w", encoding="utf-8") as file:
            json.dump(
                settings,
                file,
                ensure_ascii=False,
                indent=4
            )
    except OSError:
        return False

    return True
```

- `ensure_ascii=False` اجازه می‌دهد نویسه‌های غیرلاتین، مانند فارسی، خوانا ذخیره شوند.
- `indent=4` فایل را برای انسان خواناتر می‌کند.
- مقدار `True` یا `False` مشخص می‌کند ذخیره‌سازی موفق بوده است یا نه.

## ۵. اعتبارسنجی تنظیمات

فایل تنظیمات ممکن است به‌صورت دستی تغییر کند یا مقداری نامعتبر در آن ذخیره شود. پیش از استفاده از مقدارها، آن‌ها را بررسی کنید.

```python
def validate_settings(settings):
    geometry = settings.get("window_geometry", "720x420")

    if not isinstance(geometry, str):
        geometry = "720x420"

    if geometry not in {"720x420", "900x600", "1100x700"}:
        geometry = "720x420"

    show_tips = settings.get("show_tips", True)
    if not isinstance(show_tips, bool):
        show_tips = True

    return {
        "window_geometry": geometry,
        "show_tips": show_tips
    }
```

برای سادگی آموزشی، این مثال از چند اندازه مجاز استفاده می‌کند. در برنامه واقعی می‌توان اندازه پنجره را با قواعد دقیق‌تر اعتبارسنجی کرد.

## ۶. پروژه عملی: پنجره‌ای با تنظیمات ماندگار

کد زیر یک برنامه مستقل و قابل اجراست. اندازه پنجره و گزینه نمایش نکته‌ها در `settings.json` ذخیره می‌شوند. هنگام بستن پنجره، تنظیمات ذخیره خواهند شد.

```python
import json
import tkinter as tk
from pathlib import Path
from tkinter import ttk, messagebox


SETTINGS_FILE = Path("settings.json")

DEFAULT_SETTINGS = {
    "window_geometry": "720x420",
    "show_tips": True
}

ALLOWED_GEOMETRIES = {
    "720x420",
    "900x600",
    "1100x700"
}


def validate_settings(settings):
    geometry = settings.get("window_geometry", "720x420")

    if not isinstance(geometry, str):
        geometry = "720x420"

    if geometry not in ALLOWED_GEOMETRIES:
        geometry = "720x420"

    show_tips = settings.get("show_tips", True)

    if not isinstance(show_tips, bool):
        show_tips = True

    return {
        "window_geometry": geometry,
        "show_tips": show_tips
    }


def load_settings():
    settings = DEFAULT_SETTINGS.copy()

    try:
        with SETTINGS_FILE.open("r", encoding="utf-8") as file:
            loaded = json.load(file)

        if isinstance(loaded, dict):
            settings.update(loaded)

    except FileNotFoundError:
        pass
    except (OSError, json.JSONDecodeError):
        settings = DEFAULT_SETTINGS.copy()

    return validate_settings(settings)


def save_settings(settings):
    try:
        with SETTINGS_FILE.open("w", encoding="utf-8") as file:
            json.dump(
                settings,
                file,
                ensure_ascii=False,
                indent=4
            )
    except OSError:
        return False

    return True


class SettingsApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Application Settings")

        self.settings = load_settings()
        self.root.geometry(self.settings["window_geometry"])
        self.root.minsize(600, 350)

        self.show_tips_var = tk.BooleanVar(
            value=self.settings["show_tips"]
        )

        main_frame = ttk.Frame(root, padding=20)
        main_frame.pack(fill="both", expand=True)

        ttk.Label(
            main_frame,
            text="Application Settings",
            font=("TkDefaultFont", 16, "bold")
        ).pack(anchor="w", pady=(0, 16))

        ttk.Label(
            main_frame,
            text="Choose a window size:"
        ).pack(anchor="w")

        self.geometry_var = tk.StringVar(
            value=self.settings["window_geometry"]
        )

        geometry_box = ttk.Combobox(
            main_frame,
            textvariable=self.geometry_var,
            values=sorted(ALLOWED_GEOMETRIES),
            state="readonly"
        )
        geometry_box.pack(anchor="w", pady=(6, 14))

        ttk.Checkbutton(
            main_frame,
            text="Show helpful tips",
            variable=self.show_tips_var
        ).pack(anchor="w", pady=(0, 14))

        ttk.Button(
            main_frame,
            text="Apply Settings",
            command=self.apply_settings
        ).pack(anchor="w")

        self.status_label = ttk.Label(
            main_frame,
            text="Settings are ready."
        )
        self.status_label.pack(anchor="w", pady=(18, 0))

        self.tip_label = ttk.Label(
            main_frame,
            text="Tip: Settings will be saved when you close the app."
        )
        self.tip_label.pack(anchor="w", pady=(8, 0))

        self.update_tip_visibility()
        self.root.protocol("WM_DELETE_WINDOW", self.on_close)

    def apply_settings(self):
        geometry = self.geometry_var.get()

        if geometry not in ALLOWED_GEOMETRIES:
            messagebox.showwarning(
                "Invalid Setting",
                "Choose a valid window size."
            )
            return

        self.settings = {
            "window_geometry": geometry,
            "show_tips": self.show_tips_var.get()
        }

        self.root.geometry(geometry)
        self.update_tip_visibility()
        self.status_label.config(text="Settings applied.")

    def update_tip_visibility(self):
        if self.show_tips_var.get():
            self.tip_label.pack(anchor="w", pady=(8, 0))
        else:
            self.tip_label.pack_forget()

    def on_close(self):
        self.settings = {
            "window_geometry": self.geometry_var.get(),
            "show_tips": self.show_tips_var.get()
        }

        if save_settings(self.settings):
            self.root.destroy()
        else:
            should_close = messagebox.askyesno(
                "Save Error",
                "Settings could not be saved. Close anyway?"
            )
            if should_close:
                self.root.destroy()


if __name__ == "__main__":
    root = tk.Tk()
    app = SettingsApp(root)
    root.mainloop()
```

### روش اجرای پروژه

1. کد را در فایلی به نام `main.py` ذخیره کنید.
2. برنامه را اجرا کنید.
3. اندازه پنجره و گزینه نمایش نکته‌ها را تغییر دهید.
4. برنامه را ببندید.
5. دوباره اجرا کنید و بررسی کنید که تنظیمات قبلی بازیابی شده‌اند.
6. فایل `settings.json` را باز کنید و ساختار آن را بررسی کنید.

### نکته درباره مسیر فایل

در این مثال، `Path("settings.json")` مسیر نسبی است؛ بنابراین فایل در مسیر کاری فعلی برنامه ساخته می‌شود. اگر برنامه از محیط‌های مختلف اجرا شود، ممکن است مسیر کاری تغییر کند. برای برنامه‌های جدی‌تر بهتر است مسیر تنظیمات را در یک پوشه مشخص و قابل پیش‌بینی قرار دهید.

## ۷. نکات تجربه کاربری

- بعد از انجام عملیات، نتیجه را به کاربر اعلام کنید.
- دکمه‌ها باید نام روشن و قابل فهم داشته باشند.
- گزینه‌های انتخابی باید وضعیت فعلی را منعکس کنند.
- برای عملیات مهم، پیام تأیید در نظر بگیرید.
- اگر ذخیره تنظیمات شکست خورد، تصمیم بگیرید آیا بستن برنامه مجاز است یا خیر.
- از پیام‌های خطای بسیار فنی در رابط گرافیکی پرهیز کنید.
- برای تغییر اندازه پنجره، حداقل اندازه مناسبی تعیین کنید تا اجزای رابط از دسترس خارج نشوند.

## ۸. تمرین‌های کلاسی

### تمرین ۱ — افزودن تنظیم جدید

گزینه‌ای به نام `confirm_before_exit` اضافه کنید که تعیین کند هنگام بستن برنامه، پیام تأیید نمایش داده شود یا خیر.

**پاسخ پیشنهادی:**

به تنظیمات پیش‌فرض اضافه کنید:

```python
DEFAULT_SETTINGS = {
    "window_geometry": "720x420",
    "show_tips": True,
    "confirm_before_exit": True
}
```

سپس در `validate_settings()` مقدار را بررسی کنید:

```python
confirm_before_exit = settings.get("confirm_before_exit", True)

if not isinstance(confirm_before_exit, bool):
    confirm_before_exit = True
```

این مقدار را نیز در دیکشنری خروجی تابع برگردانید و هنگام بستن برنامه از آن استفاده کنید.

### تمرین ۲ — فایل خراب

اگر `settings.json` محتوای JSON معتبری نداشته باشد، برنامه چه رفتاری باید داشته باشد؟

**پاسخ:** خطای `json.JSONDecodeError` مدیریت شود و برنامه با تنظیمات پیش‌فرض بالا بیاید؛ بهتر است در صورت نیاز، خطا نیز ثبت شود.

### تمرین ۳ — تنظیم نامعتبر

فرض کنید فایل JSON این مقدار را داشته باشد:

```json
{
  "window_geometry": "tiny",
  "show_tips": "yes"
}
```

**پاسخ:** تابع اعتبارسنجی باید اندازه پنجره را به مقدار مجاز پیش‌فرض برگرداند و چون `"yes"` مقدار بولی واقعی نیست، `show_tips` را نیز روی `True` قرار دهد.

### تمرین ۴ — تنظیمات در SQLite یا JSON؟

برای اطلاعات دانشجو و ترجیحات ساده کاربر کدام را انتخاب می‌کنید؟

**پاسخ:** اطلاعات دانشجو در SQLite و تنظیمات ساده رابط در JSON. انتخاب نهایی به حجم، رابطه‌ها، نیازهای جست‌وجو و نحوه استفاده از داده بستگی دارد.

## ۹. خطاهای رایج

- **فرض کردن اینکه فایل همیشه وجود دارد:** اجرای اول برنامه باید بدون فایل تنظیمات نیز موفق باشد.
- **اعتماد کامل به JSON:** معتبر بودن ساختار JSON تضمین نمی‌کند مقدارها مناسب‌اند.
- **نادیده گرفتن خطای ذخیره‌سازی:** کاربر باید بداند تنظیمات ذخیره نشده‌اند.
- **استفاده از مسیر نسبی بدون آگاهی از مسیر کاری:** ممکن است فایل در پوشه‌ای غیرمنتظره ایجاد شود.
- **به‌روزرسانی تنظیمات فقط در حافظه:** برای ماندگاری، باید فایل نیز نوشته شود.
- **ذخیره همه داده‌ها در JSON:** داده‌های اصلی و مرتبط برنامه معمولاً بهتر است در پایگاه داده نگهداری شوند.

## ۱۰. چک‌لیست مدرس

- [ ] تفاوت داده‌های اصلی و تنظیمات برنامه را توضیح دادم.
- [ ] فایل JSON را بارگذاری و ذخیره کردیم.
- [ ] مقدارهای پیش‌فرض را پیاده‌سازی کردیم.
- [ ] تنظیمات را پیش از استفاده اعتبارسنجی کردیم.
- [ ] خطاهای فایل و JSON را مدیریت کردیم.
- [ ] بازیابی تنظیمات در اجرای بعدی را آزمایش کردیم.
- [ ] درباره مسیر فایل و تجربه کاربری صحبت کردیم.

## جلسه بعد

در جلسه ۱۸ می‌توانیم برنامه مدیریت دانشجو را به‌عنوان پروژه نهایی تثبیت کنیم: نیازمندی‌ها، تقسیم‌بندی صفحه‌ها، ساختار فایل‌ها، جریان کار کاربر و برنامه آزمون را مشخص کنیم.
