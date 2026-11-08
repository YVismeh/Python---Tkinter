---
title: "Tkinter - جلسه 01 - آشنایی و اولین پنجره"
aliases:
  - "Tkinter Session 01"
  - "آشنایی با رابط گرافیکی در پایتون"
tags:
  - Python
  - Tkinter
  - GUI
  - Teaching
  - Session-01
level: Beginner
status: Ready
language: Persian
---

# جلسه ۰۱ — آشنایی با Tkinter و ساخت اولین پنجره

> [!abstract] مشخصات جلسه
> - **موضوع:** آشنایی با رابط گرافیکی و ساخت اولین برنامه Tkinter
> - **زمان پیشنهادی:** ۹۰ دقیقه
> - **پیش‌نیاز:** متغیرها، شرط‌ها، توابع و ورودی/خروجی ساده در پایتون
> - **خروجی:** دانشجو بتواند پنجره بسازد، چند ویجت ساده اضافه کند و ورودی کاربر را پردازش کند.

## اهداف آموزشی

در پایان جلسه، دانشجو باید بتواند:

- [ ] تفاوت برنامه کنسولی و گرافیکی را توضیح دهد.
- [ ] ماژول `tkinter` را وارد کند.
- [ ] پنجره اصلی برنامه را با `Tk()` بسازد.
- [ ] عنوان، اندازه و رنگ پس‌زمینه پنجره را تنظیم کند.
- [ ] کاربرد `mainloop()` را توضیح دهد.
- [ ] از `Label`، `Entry` و `Button` استفاده کند.
- [ ] مقدار واردشده در `Entry` را بخواند.
- [ ] با کلیک روی دکمه یک تابع را اجرا کند.
- [ ] ورودی ساده را پردازش و نتیجه را در پنجره نمایش دهد.

---

## ۱. برنامه کنسولی و برنامه گرافیکی

در برنامه کنسولی، کاربر معمولاً اطلاعات را از طریق متن وارد می‌کند و نتیجه را نیز در محیط متنی می‌بیند.

```python
name = input("Enter your name: ")
print("Hello", name)
```

در برنامه گرافیکی (GUI)، کاربر با اجزایی مانند پنجره، دکمه، کادر ورودی و برچسب کار می‌کند.

**GUI** مخفف *Graphical User Interface* و به معنی «رابط گرافیکی کاربر» است.

نمونه‌هایی از اجزای رابط گرافیکی:

- **Window:** پنجره اصلی برنامه
- **Label:** نمایش متن
- **Entry:** دریافت یک خط متن
- **Button:** اجرای یک عملیات با کلیک کاربر

## ۲. Tkinter چیست؟

`tkinter` کتابخانه استاندارد پایتون برای ساخت رابط گرافیکی با استفاده از ابزار Tk است. در بسیاری از نصب‌های پایتون آماده استفاده است؛ در بعضی توزیع‌های لینوکس ممکن است لازم باشد بسته مربوط به Tkinter جداگانه نصب شود.

برای واردکردن کتابخانه:

```python
import tkinter as tk
```

در این مثال، `tk` نام کوتاهی است که برای دسترسی به اجزای Tkinter استفاده می‌کنیم.

## ۳. ساخت اولین پنجره

```python
import tkinter as tk

root = tk.Tk()

root.mainloop()
```

### توضیح کد

- `import tkinter as tk`: کتابخانه را وارد می‌کند.
- `tk.Tk()`: پنجره اصلی برنامه را می‌سازد.
- `root`: متغیری است که به پنجره اصلی اشاره می‌کند.
- `root.mainloop()`: حلقه رویداد را اجرا می‌کند تا پنجره باز بماند و به تعامل کاربر پاسخ دهد.

> [!important] نکته تدریسی
> در یک برنامه معمولی Tkinter، پنجره اصلی را با `Tk()` یک بار می‌سازیم و در پایان `mainloop()` را اجرا می‌کنیم. برای پنجره‌های فرعی معمولاً از `Toplevel()` استفاده می‌شود.

## ۴. تنظیم ویژگی‌های پنجره

```python
import tkinter as tk

root = tk.Tk()

root.title("My First App")
root.geometry("500x350")
root.configure(bg="#f0f0f0")
root.resizable(False, False)

root.mainloop()
```

### ویژگی‌های مهم

| دستور | کاربرد |
|---|---|
| `title("My First App")` | تعیین عنوان پنجره |
| `geometry("500x350")` | تعیین عرض و ارتفاع اولیه برحسب پیکسل |
| `configure(bg="#f0f0f0")` | تنظیم رنگ پس‌زمینه |
| `resizable(False, False)` | جلوگیری از تغییر عرض و ارتفاع پنجره |

در `geometry("500x350")` عدد اول عرض و عدد دوم ارتفاع است.

پارامترهای `resizable()` به‌ترتیب مشخص می‌کنند آیا تغییر عرض و تغییر ارتفاع مجاز است یا خیر.

## ۵. ویجت چیست؟

**Widget** به یکی از اجزای رابط گرافیکی گفته می‌شود؛ برای مثال برچسب، دکمه، کادر ورودی یا فریم.

در این جلسه سه ویجت اصلی را یاد می‌گیریم.

### ۵.۱. ویجت `Label`

برای نمایش متن یا اطلاعات کوتاه استفاده می‌شود.

```python
import tkinter as tk

root = tk.Tk()
root.title("Label Example")

label = tk.Label(root, text="Welcome to Tkinter")
label.pack()

root.mainloop()
```

در این مثال:

- `root` والد ویجت است.
- `text` متن نمایشی را مشخص می‌کند.
- `pack()` ویجت را در پنجره قرار می‌دهد.

### ۵.۲. ویجت `Entry`

برای دریافت یک خط متن از کاربر استفاده می‌شود.

```python
import tkinter as tk

root = tk.Tk()
root.title("Entry Example")

entry = tk.Entry(root)
entry.pack()

root.mainloop()
```

برای خواندن متن واردشده از متد `get()` استفاده می‌کنیم:

```python
value = entry.get()
```

مقدار برگشتی `get()` از نوع رشته (`str`) است؛ حتی اگر کاربر عدد وارد کرده باشد.

### ۵.۳. ویجت `Button`

برای اجرای یک عملیات با کلیک کاربر استفاده می‌شود.

```python
import tkinter as tk

def show_message():
    print("Button clicked")

root = tk.Tk()
root.title("Button Example")

button = tk.Button(root, text="Click Me", command=show_message)
button.pack()

root.mainloop()
```

ویژگی `command` تابعی را مشخص می‌کند که هنگام کلیک اجرا می‌شود.

> [!warning] اشتباه رایج
> از `command=show_message` استفاده می‌کنیم، نه `command=show_message()`.
>
> حالت اول، خود تابع را برای اجرای بعدی معرفی می‌کند؛ حالت دوم تابع را همان لحظه اجرا می‌کند.

## ۶. اولین برنامه تعاملی: دریافت نام کاربر

در این برنامه، کاربر نام خود را وارد می‌کند و با کلیک روی دکمه، پیام خوشامدگویی را در پنجره می‌بیند.

```python
import tkinter as tk


def greet_user():
    name = name_entry.get().strip()

    if name == "":
        result_label.config(text="Please enter your name.")
    else:
        result_label.config(text=f"Hello, {name}!")


root = tk.Tk()
root.title("Greeting App")
root.geometry("350x220")
root.resizable(False, False)

title_label = tk.Label(root, text="Enter your name:")
title_label.pack(pady=10)

name_entry = tk.Entry(root, width=25)
name_entry.pack(pady=5)

greet_button = tk.Button(
    root,
    text="Greet Me",
    command=greet_user
)
greet_button.pack(pady=10)

result_label = tk.Label(root, text="Your message will appear here.")
result_label.pack(pady=5)

root.mainloop()
```

### بررسی قسمت‌های مهم برنامه

1. تابع `greet_user()` هنگام کلیک روی دکمه اجرا می‌شود.
2. `name_entry.get()` متن واردشده را دریافت می‌کند.
3. `strip()` فاصله‌های خالی ابتدا و انتهای متن را حذف می‌کند.
4. شرط `if name == ""` بررسی می‌کند که ورودی خالی نباشد.
5. `result_label.config(text=...)` متن یک ویجت موجود را تغییر می‌دهد.
6. `pady` فاصله عمودی اطراف ویجت را مشخص می‌کند.

> [!note] نکته
> `strip()` فاصله‌های ابتدا و انتهای رشته را حذف می‌کند؛ فاصله‌های میان کلمات را حذف نمی‌کند.

## ۷. مثال دوم: جمع دو عدد

این مثال نشان می‌دهد چگونه می‌توان مقدارهای ورودی را دریافت، به عدد تبدیل و پردازش کرد.

```python
import tkinter as tk


def calculate_sum():
    try:
        first_number = float(first_entry.get())
        second_number = float(second_entry.get())

        result = first_number + second_number
        result_label.config(text=f"Result: {result}")

    except ValueError:
        result_label.config(text="Please enter valid numbers.")


root = tk.Tk()
root.title("Addition Calculator")
root.geometry("350x280")
root.resizable(False, False)

tk.Label(root, text="First number:").pack(pady=(10, 2))
first_entry = tk.Entry(root)
first_entry.pack()

tk.Label(root, text="Second number:").pack(pady=(10, 2))
second_entry = tk.Entry(root)
second_entry.pack()

calculate_button = tk.Button(
    root,
    text="Calculate",
    command=calculate_sum
)
calculate_button.pack(pady=12)

result_label = tk.Label(root, text="Result will appear here.")
result_label.pack(pady=5)

root.mainloop()
```

### نکات آموزشی مثال

- خروجی `Entry.get()` رشته است.
- `float()` رشته عددی را به عدد اعشاری تبدیل می‌کند.
- اگر ورودی قابل تبدیل به عدد نباشد، `ValueError` رخ می‌دهد.
- `try/except` مانع از توقف ناگهانی برنامه در این حالت می‌شود.
- نتیجه با تغییر متن `Label` نمایش داده می‌شود.

---

## ۸. پروژه کلاسی: فرم معرفی ساده

### صورت مسئله

برنامه‌ای طراحی کنید که نام و سن کاربر را دریافت کند. پس از کلیک روی دکمه، پیامی شامل نام و سن او نمایش دهد.

شرایط:

- اگر نام خالی باشد، پیام مناسب نمایش داده شود.
- سن باید عدد صحیح باشد.
- اگر سن معتبر نباشد، برنامه نباید متوقف شود.
- نتیجه درون پنجره نمایش داده شود.

### کد کامل پروژه

```python
import tkinter as tk


def show_information():
    name = name_entry.get().strip()
    age_text = age_entry.get().strip()

    if name == "":
        result_label.config(text="Please enter your name.")
        return

    try:
        age = int(age_text)

        if age < 0:
            result_label.config(text="Age cannot be negative.")
            return

    except ValueError:
        result_label.config(text="Please enter a valid age.")
        return

    result_label.config(
        text=f"Hello {name}. You are {age} years old."
    )


root = tk.Tk()
root.title("Personal Information")
root.geometry("380x300")
root.resizable(False, False)

tk.Label(root, text="Name:").pack(pady=(12, 2))
name_entry = tk.Entry(root, width=28)
name_entry.pack()

tk.Label(root, text="Age:").pack(pady=(12, 2))
age_entry = tk.Entry(root, width=28)
age_entry.pack()

tk.Button(
    root,
    text="Show Information",
    command=show_information
).pack(pady=15)

result_label = tk.Label(
    root,
    text="Your information will appear here.",
    wraplength=320
)
result_label.pack(pady=5)

root.mainloop()
```

## ۹. خطاهای رایج جلسه

> [!warning] خطاهای متداول
> ۱. فراموش‌کردن `mainloop()`؛ پنجره ممکن است مطابق انتظار اجرا نشود.
>
> ۲. استفاده از `command=my_function()` به‌جای `command=my_function`.
>
> ۳. تصور اینکه مقدار `Entry.get()` عدد است؛ مقدار آن رشته است.
>
> ۴. تبدیل مستقیم ورودی نامعتبر با `int()` یا `float()` بدون مدیریت خطا.
>
> ۵. تلاش برای تغییر متن یک `Label` با متغیر رشته‌ای معمولی، به‌جای استفاده از `config()` یا متغیر Tkinter.
>
> ۶. قرار دادن `pack()` و `grid()` برای ویجت‌های مختلف در یک والد مشترک؛ این دو مدیر چیدمان را نباید برای مدیریت فرزندان یک والد با هم ترکیب کرد.

---

## ۱۰. تمرین‌های دانشجو

### تمرین ۱ — پیام خوشامدگویی

برنامه‌ای بنویسید که نام کاربر را دریافت کند و با کلیک روی دکمه، پیام `Welcome, Ali!` را با نام واردشده نمایش دهد.

### تمرین ۲ — محاسبه سن

برنامه‌ای بسازید که سال تولد را دریافت کند و سن تقریبی کاربر را با فرض سال جاری ۲۰۲۶ محاسبه کند. ورودی نامعتبر باید مدیریت شود.

### تمرین ۳ — تبدیل دما

دو ورودی برای مقدار دما و یک دکمه طراحی کنید. با کلیک روی دکمه، دمای سلسیوس را به فارنهایت تبدیل کنید.

فرمول:

$$F = \frac{9}{5}C + 32
$$

### تمرین ۴ — ماشین‌حساب ساده

دو عدد دریافت کنید و با چهار دکمه، عملیات جمع، تفریق، ضرب و تقسیم را انجام دهید. خطای ورودی نامعتبر و تقسیم بر صفر را مدیریت کنید.

---

## ۱۱. پاسخ تمرین‌ها

### پاسخ تمرین ۱

```python
import tkinter as tk


def greet():
    name = name_entry.get().strip()

    if name:
        result_label.config(text=f"Welcome, {name}!")
    else:
        result_label.config(text="Please enter your name.")


root = tk.Tk()
root.title("Welcome App")
root.geometry("320x200")

name_entry = tk.Entry(root)
name_entry.pack(pady=10)

tk.Button(root, text="Welcome", command=greet).pack()

result_label = tk.Label(root, text="")
result_label.pack(pady=10)

root.mainloop()
```

### پاسخ تمرین ۲

```python
import tkinter as tk


def calculate_age():
    try:
        birth_year = int(year_entry.get())
        age = 2026 - birth_year

        if birth_year <= 0 or birth_year > 2026:
            result_label.config(text="Please enter a valid year.")
        else:
            result_label.config(text=f"Your approximate age is {age}.")

    except ValueError:
        result_label.config(text="Please enter a valid year.")


root = tk.Tk()
root.title("Age Calculator")
root.geometry("330x200")

year_entry = tk.Entry(root)
year_entry.pack(pady=10)

tk.Button(root, text="Calculate Age", command=calculate_age).pack()

result_label = tk.Label(root, text="")
result_label.pack(pady=10)

root.mainloop()
```

### پاسخ تمرین ۳

```python
import tkinter as tk


def convert_temperature():
    try:
        celsius = float(temperature_entry.get())
        fahrenheit = (9 / 5) * celsius + 32
        result_label.config(text=f"Fahrenheit: {fahrenheit:.2f}")

    except ValueError:
        result_label.config(text="Please enter a valid temperature.")


root = tk.Tk()
root.title("Temperature Converter")
root.geometry("350x200")

tk.Label(root, text="Temperature in Celsius:").pack(pady=5)

temperature_entry = tk.Entry(root)
temperature_entry.pack()

tk.Button(
    root,
    text="Convert",
    command=convert_temperature
).pack(pady=10)

result_label = tk.Label(root, text="")
result_label.pack()

root.mainloop()
```

### پاسخ تمرین ۴

```python
import tkinter as tk


def calculate(operation):
    try:
        first_number = float(first_entry.get())
        second_number = float(second_entry.get())

        if operation == "+":
            result = first_number + second_number
        elif operation == "-":
            result = first_number - second_number
        elif operation == "*":
            result = first_number * second_number
        elif operation == "/":
            if second_number == 0:
                result_label.config(text="Cannot divide by zero.")
                return
            result = first_number / second_number

        result_label.config(text=f"Result: {result}")

    except ValueError:
        result_label.config(text="Please enter valid numbers.")


root = tk.Tk()
root.title("Simple Calculator")
root.geometry("350x300")

first_entry = tk.Entry(root)
first_entry.pack(pady=5)

second_entry = tk.Entry(root)
second_entry.pack(pady=5)

button_frame = tk.Frame(root)
button_frame.pack(pady=10)

for symbol in ["+", "-", "*", "/"]:
    tk.Button(
        button_frame,
        text=symbol,
        width=5,
        command=lambda op=symbol: calculate(op)
    ).pack(side="left", padx=3)

result_label = tk.Label(root, text="Result will appear here.")
result_label.pack(pady=10)

root.mainloop()
```

> [!tip] نکته مدرس
> در تمرین چهارم، `lambda op=symbol` باعث می‌شود هر دکمه عملیات مربوط به خودش را ارسال کند. این موضوع را می‌توان به‌عنوان نکته تکمیلی توضیح داد؛ اگر دانشجو هنوز با `lambda` آشنا نیست، ابتدا نسخه‌ای با چهار تابع جداگانه آموزش بده.

---

## ۱۲. جمع‌بندی جلسه

مهم‌ترین مفاهیم این جلسه:

| مفهوم | کاربرد |
|---|---|
| `Tk()` | ساخت پنجره اصلی |
| `mainloop()` | اجرای حلقه رویداد |
| `title()` | تعیین عنوان پنجره |
| `geometry()` | تعیین ابعاد اولیه |
| `Label` | نمایش متن |
| `Entry` | دریافت یک خط متن |
| `Button` | اجرای عملیات با کلیک |
| `command` | اتصال تابع به دکمه |
| `get()` | دریافت متن ورودی |
| `config()` | تغییر ویژگی‌های ویجت |
| `pack()` | قرار دادن ویجت در چیدمان |
| `strip()` | حذف فاصله‌های ابتدا و انتهای رشته |
| `try/except` | مدیریت خطاهای ورودی |

## چک‌لیست پایان جلسه

- [ ] دانشجو می‌تواند پنجره Tkinter ایجاد کند.
- [ ] می‌تواند عنوان و ابعاد پنجره را تنظیم کند.
- [ ] می‌تواند `Label`، `Entry` و `Button` بسازد.
- [ ] می‌تواند مقدار ورودی را دریافت کند.
- [ ] می‌تواند با کلیک روی دکمه یک تابع را اجرا کند.
- [ ] می‌تواند نتیجه را در رابط گرافیکی نمایش دهد.
- [ ] می‌تواند ورودی نامعتبر را مدیریت کند.

## جلسه بعد

[[Tkinter - جلسه 02 - ویجت‌های پایه و مدیریت چیدمان]]

در جلسه بعد، ویجت‌های پایه را عمیق‌تر بررسی می‌کنیم، متدهای پرکاربرد آن‌ها را یاد می‌گیریم و به سراغ مدیریت چیدمان با `pack()` و `grid()`، ساخت `Frame` و `LabelFrame` و طراحی فرم‌های منظم می‌رویم.
