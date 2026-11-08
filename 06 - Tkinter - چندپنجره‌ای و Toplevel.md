---
title: "Tkinter - جلسه 06 - چندپنجره‌ای و Toplevel"
aliases:
  - "Tkinter Session 6"
  - "مدیریت چند پنجره در Tkinter"
tags:
  - Python
  - Tkinter
  - GUI
  - Teaching
level: Beginner-to-Intermediate
status: Ready
language: Persian
session: 6
---

# جلسه ۶ — ساخت برنامه‌های چندپنجره‌ای با `Toplevel`

> [!abstract] هدف جلسه
> در این جلسه یاد می‌گیریم چطور در یک برنامه چند پنجره داشته باشیم، پنجره اصلی را از پنجره‌های فرعی جدا کنیم، اطلاعات را بین پنجره‌ها جابه‌جا کنیم و از ابزارهای `ttk` برای ساخت رابط کاربری یکدست‌تر استفاده کنیم.

## ۱. چرا به چند پنجره نیاز داریم؟

در برنامه‌های واقعی، تمام امکانات را در یک پنجره قرار نمی‌دهیم. برای نمونه:

- پنجره اصلی برنامه
- پنجره تنظیمات
- پنجره ثبت اطلاعات
- پنجره درباره برنامه
- پنجره نمایش جزئیات یک مورد

برای چنین برنامه‌هایی می‌توان از `Toplevel` استفاده کرد.

## ۲. تفاوت `Tk()` و `Toplevel`

- `Tk()` معمولاً برای ساخت پنجره اصلی برنامه استفاده می‌شود.
- `Toplevel(parent)` پنجره‌ای جدید در همان برنامه ایجاد می‌کند.
- در یک برنامه معمولی Tkinter بهتر است فقط یک نمونه اصلی `Tk()` داشته باشیم و برای پنجره‌های دیگر از `Toplevel` استفاده کنیم.

ساخت چند نمونه `Tk()` می‌تواند باعث ایجاد چند مفسر Tcl/Tk و رفتارهای پیچیده شود؛ برای پنجره‌های فرعی از `Toplevel` استفاده کنید.

## ۳. اولین پنجره فرعی

```python
import tkinter as tk

root = tk.Tk()
root.title("Main Window")
root.geometry("350x220")

def open_child_window():
    child = tk.Toplevel(root)
    child.title("Child Window")
    child.geometry("300x160")

    tk.Label(child, text="This is a secondary window.").pack(pady=20)
    tk.Button(child, text="Close", command=child.destroy).pack(pady=5)

tk.Label(root, text="Main Application", font=("Arial", 14)).pack(pady=20)
tk.Button(root, text="Open Window", command=open_child_window).pack(pady=10)

root.mainloop()
```

هر بار کلیک روی دکمه، یک پنجره جدید ساخته می‌شود. در برنامه‌های بزرگ‌تر معمولاً لازم است از باز شدن چندباره یک پنجره تنظیمات یا فرم جلوگیری کنیم.

## ۴. جلوگیری از باز شدن چندباره یک پنجره

یکی از روش‌های ساده، نگه داشتن مرجع پنجره فرعی و بررسی زنده بودن آن است.

```python
import tkinter as tk

root = tk.Tk()
root.title("Single Child Window")
root.geometry("350x200")

child_window = None

def open_child_window():
    global child_window

    if child_window is not None and child_window.winfo_exists():
        child_window.lift()
        child_window.focus_force()
        return

    child_window = tk.Toplevel(root)
    child_window.title("Settings")
    child_window.geometry("280x160")

    tk.Label(child_window, text="Settings Window").pack(pady=20)
    tk.Button(child_window, text="Close", command=child_window.destroy).pack()

tk.Button(root, text="Open Settings", command=open_child_window).pack(pady=50)

root.mainloop()
```

این الگو برای شروع قابل فهم است. در پروژه‌های بزرگ‌تر می‌توان مرجع پنجره را هنگام بسته شدن پاک کرد یا پنجره را در یک کلاس مدیریت کرد. استفاده از `winfo_exists()` کمک می‌کند پیش از استفاده از مرجع، وجود پنجره بررسی شود.

## ۵. انتقال اطلاعات از پنجره اصلی به پنجره فرعی

فرض کنیم کاربر نام خود را در پنجره اصلی وارد می‌کند و با کلیک روی دکمه، پنجره جدیدی باز می‌شود که همان نام را نمایش می‌دهد.

```python
import tkinter as tk

root = tk.Tk()
root.title("Pass Data to Child")
root.geometry("350x200")

name_entry = tk.Entry(root)
name_entry.pack(pady=12)

def open_greeting_window():
    name = name_entry.get().strip()

    child = tk.Toplevel(root)
    child.title("Greeting")
    child.geometry("280x140")

    if name:
        message = f"Hello, {name}!"
    else:
        message = "Hello, guest!"

    tk.Label(child, text=message, font=("Arial", 12)).pack(pady=25)

tk.Label(root, text="Enter your name:").pack(pady=5)
name_entry.pack(pady=8)

tk.Button(root, text="Open Greeting", command=open_greeting_window).pack(pady=8)

root.mainloop()
```

در این مثال، تابع `open_greeting_window()` مقدار `Entry` را می‌خواند و آن را به پنجره جدید می‌دهد. این روش برای اطلاعات ساده مناسب است.

## ۶. انتقال اطلاعات با متغیرهای Tkinter

اگر هر دو پنجره به یک متغیر `StringVar` متصل باشند، تغییر مقدار متغیر می‌تواند در هر دو پنجره دیده شود.

```python
import tkinter as tk

root = tk.Tk()
root.title("Shared Variable")
root.geometry("350x220")

shared_name = tk.StringVar(value="Student")

tk.Label(root, text="Name in main window:").pack(pady=5)
tk.Entry(root, textvariable=shared_name).pack(padx=15, fill="x")

def open_preview():
    child = tk.Toplevel(root)
    child.title("Preview")
    child.geometry("300x140")

    tk.Label(child, text="Live preview:").pack(pady=8)
    tk.Label(child, textvariable=shared_name, font=("Arial", 14, "bold")).pack(pady=8)

tk.Button(root, text="Open Preview", command=open_preview).pack(pady=12)

root.mainloop()
```

در این روش، متغیر `shared_name` بین دو پنجره مشترک است؛ بنابراین با تغییر متن ورودی اصلی، متن پنجره پیش‌نمایش نیز به‌روزرسانی می‌شود.

## ۷. پنجره مودال با `transient()` و `grab_set()`

گاهی می‌خواهیم تا وقتی یک پنجره فرعی باز است، کاربر نتواند با پنجره اصلی تعامل کند. این رفتار را می‌توان با یک پنجره مودال ساده ایجاد کرد.

```python
import tkinter as tk

root = tk.Tk()
root.title("Modal Window Example")
root.geometry("350x200")

def open_modal():
    dialog = tk.Toplevel(root)
    dialog.title("Confirmation")
    dialog.geometry("280x150")
    dialog.transient(root)
    dialog.grab_set()

    tk.Label(dialog, text="Complete this dialog first.").pack(pady=20)
    tk.Button(dialog, text="Close", command=dialog.destroy).pack(pady=5)

tk.Label(root, text="Main Window").pack(pady=25)
tk.Button(root, text="Open Dialog", command=open_modal).pack()

root.mainloop()
```

- `transient(root)` پنجره فرعی را از نظر ارتباط پنجره‌ای به پنجره اصلی مرتبط می‌کند.
- `grab_set()` رویدادهای ورودی را تا پایان مودال به آن پنجره محدود می‌کند.
- با `dialog.destroy()` پنجره بسته می‌شود و تعامل با پنجره اصلی از سر گرفته می‌شود.

برای یک پنجره مودال ساده، این روش کافی است؛ در برنامه‌های کامل‌تر باید به بستن پنجره با دکمه بستن سیستم‌عامل نیز توجه کرد.

## ۸. استفاده از `ttk`

ماژول `tkinter.ttk` مجموعه‌ای از ویجت‌های با ظاهر تم‌پذیرتر ارائه می‌کند. از جمله:

- `ttk.Button`
- `ttk.Label`
- `ttk.Entry`
- `ttk.Frame`
- `ttk.LabelFrame`
- `ttk.Combobox`
- `ttk.Checkbutton`
- `ttk.Radiobutton`

مثال:

```python
import tkinter as tk
from tkinter import ttk

root = tk.Tk()
root.title("TTK Widgets")
root.geometry("360x220")

main_frame = ttk.Frame(root, padding=15)
main_frame.pack(fill="both", expand=True)

ttk.Label(main_frame, text="Student Form").grid(row=0, column=0, columnspan=2, pady=10)

ttk.Label(main_frame, text="Name:").grid(row=1, column=0, sticky="w", padx=5, pady=5)

name_entry = ttk.Entry(main_frame)
name_entry.grid(row=1, column=1, sticky="ew", padx=5, pady=5)

ttk.Button(
    main_frame,
    text="Show Name",
    command=lambda: result_label.config(text=name_entry.get().strip())
).grid(row=2, column=0, columnspan=2, pady=10)

result_label = ttk.Label(main_frame, text="")
result_label.grid(row=3, column=0, columnspan=2, pady=5)

main_frame.columnconfigure(1, weight=1)

root.mainloop()
```

نکته: گزینه‌های ظاهری `ttk` با ویجت‌های کلاسیک `tkinter` کاملاً یکسان نیستند. برای مثال، ظاهر دکمه `ttk.Button` را معمولاً با `ttk.Style` تنظیم می‌کنیم، نه با گزینه‌هایی مثل `bg` و `fg`.

## ۹. تنظیم سبک با `ttk.Style`

```python
import tkinter as tk
from tkinter import ttk

root = tk.Tk()
root.title("TTK Style")
root.geometry("350x180")

style = ttk.Style(root)
style.configure("Accent.TButton", padding=8, font=("Arial", 11, "bold"))
style.configure("Title.TLabel", font=("Arial", 15, "bold"))

ttk.Label(root, text="Styled Interface", style="Title.TLabel").pack(pady=15)
ttk.Button(root, text="Continue", style="Accent.TButton").pack(pady=10)

root.mainloop()
```

`ttk.Style` برای تنظیم ویژگی‌هایی مانند فونت، فاصله داخلی و برخی رنگ‌ها استفاده می‌شود. نتیجه دقیق بعضی تنظیمات به تم فعال و سیستم‌عامل وابسته است.

## ۱۰. پروژه کلاسی: برنامه مدیریت دانشجو با دو پنجره

### امکانات

- پنجره اصلی برای وارد کردن نام و رشته
- پنجره فرعی برای نمایش خلاصه اطلاعات
- پنجره فرعی با ظاهر `ttk`
- دکمه پاک کردن فرم
- اعتبارسنجی نام خالی
- جلوگیری از باز شدن چندباره پنجره جزئیات

```python
import tkinter as tk
from tkinter import ttk, messagebox

root = tk.Tk()
root.title("Student Manager")
root.geometry("430x300")
root.resizable(False, False)

name_var = tk.StringVar()
major_var = tk.StringVar(value="Software")
details_window = None

main_frame = ttk.Frame(root, padding=18)
main_frame.pack(fill="both", expand=True)

ttk.Label(main_frame, text="Student Information", font=("Arial", 15, "bold")).grid(
    row=0, column=0, columnspan=2, pady=(0, 15)
)

ttk.Label(main_frame, text="Name:").grid(row=1, column=0, sticky="w", padx=5, pady=7)
name_entry = ttk.Entry(main_frame, textvariable=name_var)
name_entry.grid(row=1, column=1, sticky="ew", padx=5, pady=7)

ttk.Label(main_frame, text="Major:").grid(row=2, column=0, sticky="w", padx=5, pady=7)
major_combo = ttk.Combobox(
    main_frame,
    textvariable=major_var,
    values=["Software", "Networking", "Database", "IT"],
    state="readonly"
)
major_combo.grid(row=2, column=1, sticky="ew", padx=5, pady=7)

main_frame.columnconfigure(1, weight=1)

def show_details():
    global details_window

    name = name_var.get().strip()

    if not name:
        messagebox.showwarning("Missing Name", "Enter the student's name.")
        name_entry.focus_set()
        return

    if details_window is not None and details_window.winfo_exists():
        details_window.lift()
        details_window.focus_force()
        return

    details_window = tk.Toplevel(root)
    details_window.title("Student Details")
    details_window.geometry("320x200")
    details_window.transient(root)

    detail_frame = ttk.Frame(details_window, padding=18)
    detail_frame.pack(fill="both", expand=True)

    ttk.Label(detail_frame, text="Student Details", font=("Arial", 13, "bold")).pack(pady=(0, 12))
    ttk.Label(detail_frame, text=f"Name: {name}").pack(anchor="w", pady=4)
    ttk.Label(detail_frame, text=f"Major: {major_var.get()}").pack(anchor="w", pady=4)
    ttk.Button(detail_frame, text="Close", command=details_window.destroy).pack(pady=12)

def clear_form():
    name_var.set("")
    major_var.set("Software")
    name_entry.focus_set()

button_frame = ttk.Frame(main_frame)
button_frame.grid(row=3, column=0, columnspan=2, pady=18)

ttk.Button(button_frame, text="Show Details", command=show_details).grid(row=0, column=0, padx=5)
ttk.Button(button_frame, text="Clear Form", command=clear_form).grid(row=0, column=1, padx=5)

ttk.Label(
    main_frame,
    text="Enter the information and open the details window.",
    wraplength=370
).grid(row=4, column=0, columnspan=2, pady=5)

root.mainloop()
```

### تحلیل پروژه

1. `Tk()` پنجره اصلی را می‌سازد.
2. `Toplevel(root)` پنجره جزئیات را ایجاد می‌کند.
3. `StringVar` مقدار نام و رشته را نگه می‌دارد.
4. پیش از باز کردن پنجره، نام بررسی می‌شود.
5. متغیر `details_window` مرجع پنجره فرعی را نگه می‌دارد تا از ایجاد چند پنجره مشابه جلوگیری شود.
6. `ttk.Frame` و سایر ویجت‌های `ttk` ظاهر رابط را یکدست‌تر می‌کنند.
7. دکمه پاک کردن، ورودی‌ها را به حالت اولیه برمی‌گرداند.

## ۱۱. خطاهای رایج

### خطای ۱: ساخت یک `Tk()` برای هر پنجره

برای پنجره‌های فرعی از `Toplevel` استفاده کنید، نه ساخت چند پنجره مستقل با `Tk()`.

### خطای ۲: استفاده از متغیر محلی برای پنجره‌ای که بعداً لازم است

اگر لازم است بعداً پنجره را بررسی یا کنترل کنید، مرجع آن را در متغیری نگه دارید که خارج از تابع نیز قابل دسترسی باشد یا از کلاس استفاده کنید.

### خطای ۳: مخلوط کردن `pack()` و `grid()` در یک والد

می‌توان در یک برنامه از هر دو روش استفاده کرد، اما برای فرزندهای یک والد مشخص، آن‌ها را با هم مخلوط نکنید. برای مثال، اگر ویجت‌های مستقیم `main_frame` با `grid()` چیده شده‌اند، ویجت مستقیم دیگری را با `pack()` داخل همان `main_frame` قرار ندهید. استفاده از `pack()` در یک `Frame` داخلی و `grid()` در والد آن مشکلی ندارد.

### خطای ۴: استفاده از `bg` و `fg` برای تمام ویجت‌های `ttk`

ویجت‌های `ttk` از سیستم تم استفاده می‌کنند؛ برای سفارشی‌سازی ظاهر آن‌ها معمولاً `ttk.Style` مناسب‌تر است.

### خطای ۵: فراموش کردن اعتبارسنجی پیش از نمایش جزئیات

حتی در پنجره دوم، داده‌ها باید با دقت نمایش داده شوند و فیلدهای ضروری پیش از ادامه بررسی شوند.

## ۱۲. تمرین‌های دانشجو

### تمرین ۱ — پنجره درباره برنامه

یک پنجره اصلی با دکمه `About` بسازید. با کلیک روی آن یک `Toplevel` باز شود و نام برنامه و شماره نسخه را نشان دهد. با کلیک دوباره، پنجره تکراری ایجاد نشود.

### تمرین ۲ — انتقال پیام

در پنجره اصلی یک `Entry` قرار دهید. با کلیک روی دکمه، متن واردشده را به پنجره فرعی منتقل کنید. اگر ورودی خالی است، هشدار نمایش دهید.

### تمرین ۳ — پنجره مودال

یک پنجره فرعی با `transient()` و `grab_set()` بسازید. تا وقتی پنجره فرعی باز است، کاربر نتواند با پنجره اصلی کار کند.

### تمرین ۴ — تبدیل رابط به `ttk`

یک فرم ساده که از `Label`، `Entry` و `Button` کلاسیک استفاده می‌کند، به نسخه‌ای با `ttk.Label`، `ttk.Entry` و `ttk.Button` تبدیل کنید. برای فونت دکمه از `ttk.Style` استفاده کنید.

---

## ۱۳. پاسخ تمرین‌ها

### پاسخ تمرین ۱

```python
import tkinter as tk
from tkinter import ttk

root = tk.Tk()
root.title("My Application")
root.geometry("320x180")

about_window = None

def open_about():
    global about_window

    if about_window is not None and about_window.winfo_exists():
        about_window.lift()
        about_window.focus_force()
        return

    about_window = tk.Toplevel(root)
    about_window.title("About")
    about_window.geometry("260x140")

    ttk.Label(about_window, text="Student Manager", font=("Arial", 12, "bold")).pack(pady=15)
    ttk.Label(about_window, text="Version 1.0").pack(pady=5)
    ttk.Button(about_window, text="Close", command=about_window.destroy).pack(pady=8)

ttk.Button(root, text="About", command=open_about).pack(pady=60)

root.mainloop()
```

### پاسخ تمرین ۲

```python
import tkinter as tk
from tkinter import ttk, messagebox

root = tk.Tk()
root.title("Pass Message")
root.geometry("320x180")

message_entry = ttk.Entry(root)
message_entry.pack(fill="x", padx=15, pady=15)

def show_message():
    message = message_entry.get().strip()

    if not message:
        messagebox.showwarning("Missing Message", "Enter a message first.")
        return

    child = tk.Toplevel(root)
    child.title("Message")
    child.geometry("280x130")

    ttk.Label(child, text=message, wraplength=240).pack(padx=15, pady=25)

ttk.Button(root, text="Open Message", command=show_message).pack(pady=8)

root.mainloop()
```

### پاسخ تمرین ۳

```python
import tkinter as tk
from tkinter import ttk

root = tk.Tk()
root.title("Modal Example")
root.geometry("320x180")

def open_dialog():
    dialog = tk.Toplevel(root)
    dialog.title("Dialog")
    dialog.geometry("260x140")
    dialog.transient(root)
    dialog.grab_set()

    ttk.Label(dialog, text="Close this dialog first.").pack(pady=20)
    ttk.Button(dialog, text="Close", command=dialog.destroy).pack(pady=5)

ttk.Button(root, text="Open Dialog", command=open_dialog).pack(pady=60)

root.mainloop()
```

### پاسخ تمرین ۴

```python
import tkinter as tk
from tkinter import ttk

root = tk.Tk()
root.title("TTK Form")
root.geometry("350x200")

style = ttk.Style(root)
style.configure("Form.TButton", font=("Arial", 11, "bold"), padding=6)

main_frame = ttk.Frame(root, padding=15)
main_frame.pack(fill="both", expand=True)

name_var = tk.StringVar()
result_var = tk.StringVar()

ttk.Label(main_frame, text="Name:").grid(row=0, column=0, padx=5, pady=8)
ttk.Entry(main_frame, textvariable=name_var).grid(row=0, column=1, padx=5, pady=8)

def show_name():
    result_var.set(f"Name: {name_var.get().strip()}")

ttk.Button(
    main_frame,
    text="Show",
    style="Form.TButton",
    command=show_name
).grid(row=1, column=0, columnspan=2, pady=8)

ttk.Label(main_frame, textvariable=result_var).grid(row=2, column=0, columnspan=2, pady=5)

main_frame.columnconfigure(1, weight=1)
root.mainloop()
```

## ۱۴. جمع‌بندی و چک‌لیست

- [ ] تفاوت `Tk()` و `Toplevel` را می‌داند.
- [ ] پنجره فرعی ایجاد و بسته می‌کند.
- [ ] اطلاعات را از پنجره اصلی به پنجره فرعی می‌فرستد.
- [ ] از `StringVar` مشترک برای نمایش زنده استفاده می‌کند.
- [ ] مفهوم پنجره مودال و `grab_set()` را می‌شناسد.
- [ ] ویجت‌های پایه `ttk` را استفاده می‌کند.
- [ ] با `ttk.Style` ظاهر ساده‌ای تنظیم می‌کند.
- [ ] از باز شدن پنجره تکراری جلوگیری می‌کند.
- [ ] از `pack()` و `grid()` به‌صورت درست در والدهای جدا استفاده می‌کند.

## ۱۵. آمادگی برای جلسه بعد

در جلسه هفتم سراغ منوها، انتخاب فایل و ویرایش متن می‌رویم: `Menu`، `filedialog`، ویجت `Text` و ساخت یک ویرایشگر متن ساده.

پیوند پیشنهادی در Obsidian: `[[Tkinter - جلسه 07 - منوها و ویرایشگر متن]]`
