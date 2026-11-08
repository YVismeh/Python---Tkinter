---
title: "Tkinter - جلسه 02 - کار با ورودی‌ها، چیدمان و توابع"
aliases:
  - "Tkinter Session 2"
  - "جلسه دوم آموزش Tkinter"
tags:
  - Python
  - Tkinter
  - GUI
  - Teaching
level: Beginner
status: Ready
language: Persian
session: 2
---

# جلسه ۲ — کار با ورودی‌ها، چیدمان و توابع

> [!abstract] هدف جلسه
> در این جلسه یاد می‌گیریم اطلاعات واردشده در `Entry` را دریافت و ویرایش کنیم، با کلیک روی دکمه‌ها توابع را اجرا کنیم، خطاهای ساده را مدیریت کنیم و با `pack` و `grid` رابط کاربری مرتب‌تری بسازیم.

## ۱. پیش‌نیازها

دانشجو باید با موارد زیر آشنا باشد:

- ساخت پنجره با `Tk()`
- تنظیم عنوان و اندازه پنجره
- استفاده از `Label`، `Entry` و `Button`
- تعریف تابع در پایتون
- آشنایی مقدماتی با `try` و `except`

## ۲. مرور سریع جلسه قبل

```python
import tkinter as tk

root = tk.Tk()
root.title("My First App")
root.geometry("400x250")

label = tk.Label(root, text="Hello")
label.pack()

root.mainloop()
```

نکته‌ها:

- `Tk()` پنجره اصلی برنامه را می‌سازد.
- `pack()` ویجت را در پنجره قرار می‌دهد.
- `mainloop()` رویدادهای رابط گرافیکی را مدیریت می‌کند و پنجره را فعال نگه می‌دارد.
- بهتر است در برنامه معمولی فقط یک نمونه از `Tk()` داشته باشیم.

## ۳. دریافت اطلاعات از `Entry`

ویجت `Entry` برای دریافت یک خط متن از کاربر استفاده می‌شود.

```python
import tkinter as tk

root = tk.Tk()
root.title("Entry Example")
root.geometry("350x180")

name_label = tk.Label(root, text="Enter your name:")
name_label.pack(pady=5)

name_entry = tk.Entry(root)
name_entry.pack(pady=5)

def show_name():
    name = name_entry.get()
    result_label.config(text=f"Hello, {name}!")

result_label = tk.Label(root, text="")
result_label.pack(pady=10)

show_button = tk.Button(root, text="Show", command=show_name)
show_button.pack()

root.mainloop()
```

### متدهای مهم `Entry`

| متد | کاربرد |
|---|---|
| `get()` | دریافت متن فعلی |
| `insert(index, text)` | درج متن در موقعیت مشخص |
| `delete(first, last)` | حذف بخشی از متن |
| `config(...)` | تغییر تنظیمات ویجت |

### مثال `insert()` و `delete()`

```python
import tkinter as tk

root = tk.Tk()
root.title("Entry Methods")
root.geometry("350x180")

entry = tk.Entry(root)
entry.pack(pady=10)

entry.insert(0, "Python")

def add_text():
    entry.insert(tk.END, " Tkinter")

def clear_text():
    entry.delete(0, tk.END)

tk.Button(root, text="Add Text", command=add_text).pack(pady=3)
tk.Button(root, text="Clear", command=clear_text).pack(pady=3)

root.mainloop()
```

توضیح:

- `insert(0, "Python")` متن را از ابتدای ورودی درج می‌کند.
- `tk.END` به انتهای متن اشاره می‌کند.
- `delete(0, tk.END)` تمام متن ورودی را پاک می‌کند.

## ۴. اتصال دکمه به تابع با `command`

در Tkinter معمولاً تابع دکمه را بدون پرانتز به `command` می‌دهیم:

```python
button = tk.Button(root, text="Click", command=my_function)
```

در این حالت تابع هنگام کلیک اجرا می‌شود.

اشتباه رایج:

```python
button = tk.Button(root, text="Click", command=my_function())
```

در این حالت تابع هنگام ساخت دکمه اجرا می‌شود و مقدار بازگشتی آن به `command` داده می‌شود؛ معمولاً این چیزی نیست که می‌خواهیم.

اگر لازم است به تابع مقدار ثابت بدهیم، می‌توان از `lambda` استفاده کرد:

```python
button = tk.Button(root, text="Say Hello", command=lambda: greet("Student"))
```

در این مثال، `greet("Student")` فقط هنگام کلیک اجرا می‌شود.

## ۵. مدیریت ورودی خالی و خطاهای ساده

ورودی `Entry` همیشه رشته برمی‌گرداند؛ حتی اگر کاربر عدد وارد کند.

```python
value = entry.get()
```

برای انجام محاسبات باید رشته عددی را به نوع عددی تبدیل کنیم:

```python
number = float(value)
```

اگر متن ورودی عدد معتبر نباشد، تبدیل خطا می‌دهد. از `try/except` استفاده می‌کنیم:

```python
import tkinter as tk
from tkinter import messagebox

root = tk.Tk()
root.title("Safe Number Input")
root.geometry("350x200")

number_entry = tk.Entry(root)
number_entry.pack(pady=10)

result_label = tk.Label(root, text="Result will appear here")
result_label.pack(pady=5)

def calculate_square():
    raw_value = number_entry.get().strip()

    if raw_value == "":
        messagebox.showwarning("Missing Input", "Please enter a number.")
        return

    try:
        number = float(raw_value)
        result_label.config(text=f"Square: {number ** 2:g}")
    except ValueError:
        messagebox.showerror("Invalid Input", "Please enter a valid number.")

tk.Button(root, text="Calculate Square", command=calculate_square).pack(pady=8)

root.mainloop()
```

نکته‌های آموزشی:

- `strip()` فاصله‌های ابتدا و انتهای رشته را حذف می‌کند.
- `return` اجرای تابع را همان‌جا متوقف می‌کند.
- `messagebox.showwarning()` پیام هشدار نشان می‌دهد.
- `messagebox.showerror()` پیام خطا نشان می‌دهد.
- `ValueError` هنگام نامعتبر بودن تبدیل رشته به عدد رخ می‌دهد.

## ۶. چیدمان با `pack`

`pack()` ویجت‌ها را با توجه به فضای موجود کنار هم یا زیر هم قرار می‌دهد.

```python
label.pack()
```

پارامترهای پرکاربرد:

| پارامتر | کاربرد |
|---|---|
| `side` | جهت قرارگیری: `tk.TOP`، `tk.BOTTOM`، `tk.LEFT` یا `tk.RIGHT` |
| `fill` | پر کردن فضا در جهت `tk.X`، `tk.Y` یا `tk.BOTH` |
| `expand` | اجازه استفاده از فضای اضافی |
| `padx` | فاصله افقی بیرونی |
| `pady` | فاصله عمودی بیرونی |
| `ipadx` | فضای داخلی افقی |
| `ipady` | فضای داخلی عمودی |
| `anchor` | جهت قرارگیری ویجت در فضای تخصیص‌یافته |

مثال:

```python
import tkinter as tk

root = tk.Tk()
root.title("Pack Example")
root.geometry("400x250")

top_label = tk.Label(root, text="Top", bg="lightblue")
top_label.pack(side=tk.TOP, fill=tk.X, padx=10, pady=5)

left_label = tk.Label(root, text="Left", bg="lightgreen")
left_label.pack(side=tk.LEFT, fill=tk.Y, padx=10, pady=5)

center_label = tk.Label(root, text="Center", bg="lightyellow")
center_label.pack(expand=True, padx=10, pady=10)

root.mainloop()
```

### تفاوت `fill` و `expand`

- `fill` تعیین می‌کند ویجت در فضای اختصاص‌یافته در چه جهتی کشیده شود.
- `expand=True` اجازه می‌دهد ویجت سهمی از فضای اضافی والد بگیرد.
- ترکیب `fill="both"` و `expand=True` برای فریم‌هایی که باید با پنجره رشد کنند بسیار رایج است.

```python
main_frame.pack(fill="both", expand=True)
```

در این دستور، فریم می‌تواند هم در عرض و هم در ارتفاع بزرگ شود و فضای اضافی پنجره را دریافت کند.

## ۷. چیدمان با `grid`

`grid()` ویجت‌ها را در ردیف و ستون قرار می‌دهد و برای فرم‌ها بسیار مناسب است.

```python
label.grid(row=0, column=0)
entry.grid(row=0, column=1)
```

پارامترهای پرکاربرد:

| پارامتر | کاربرد |
|---|---|
| `row` | شماره ردیف |
| `column` | شماره ستون |
| `padx`، `pady` | فاصله بیرونی |
| `ipadx`، `ipady` | فاصله داخلی |
| `sticky` | چسبیدن به جهت‌های مشخص |
| `columnspan` | اشغال چند ستون |
| `rowspan` | اشغال چند ردیف |

مقادیر `sticky` از حروف جهت استفاده می‌کنند:

- `"w"`: چپ در رابط‌های چپ‌به‌راست
- `"e"`: راست
- `"n"`: بالا
- `"s"`: پایین
- `"ew"`: کشیده‌شدن افقی
- `"nsew"`: کشیده‌شدن در همه جهت‌ها

مثال فرم:

```python
import tkinter as tk

root = tk.Tk()
root.title("Grid Form")
root.geometry("400x180")

tk.Label(root, text="First name:").grid(row=0, column=0, padx=10, pady=8, sticky="w")
first_name_entry = tk.Entry(root)
first_name_entry.grid(row=0, column=1, padx=10, pady=8, sticky="ew")

tk.Label(root, text="Last name:").grid(row=1, column=0, padx=10, pady=8, sticky="w")
last_name_entry = tk.Entry(root)
last_name_entry.grid(row=1, column=1, padx=10, pady=8, sticky="ew")

root.columnconfigure(1, weight=1)

root.mainloop()
```

`root.columnconfigure(1, weight=1)` باعث می‌شود ستون شماره ۱ در تقسیم فضای افقی اضافی سهم بگیرد. برای کشیده‌شدن خود ویجت در آن فضا، معمولاً `sticky="ew"` را هم تنظیم می‌کنیم.

## ۸. قانون مهم: `pack` و `grid` را در یک والد ترکیب نکنید

در یک پنجره یا فریم مشخص، برای مدیریت ویجت‌های فرزند همان والد، از `pack` یا `grid` استفاده کنید؛ هر دو را با هم روی فرزندان یک والد به کار نبرید.

الگوی نامناسب:

```python
label.pack()
entry.grid(row=0, column=0)
```

الگوی مناسب: از یک روش برای فرزندان همان والد استفاده کنید، یا آن‌ها را در فریم‌های جداگانه قرار دهید و برای هر فریم روش مستقلی به کار ببرید.

```python
import tkinter as tk

root = tk.Tk()
root.title("Separate Layout Managers")
root.geometry("350x180")

top_frame = tk.Frame(root)
top_frame.pack(fill="x", padx=10, pady=10)

tk.Label(top_frame, text="Name:").pack(side="left")
tk.Entry(top_frame).pack(side="left", fill="x", expand=True)

button_frame = tk.Frame(root)
button_frame.pack(pady=10)

tk.Button(button_frame, text="Save").grid(row=0, column=0, padx=5)
tk.Button(button_frame, text="Cancel").grid(row=0, column=1, padx=5)

root.mainloop()
```

در این مثال، فرزندان `top_frame` از `pack` استفاده می‌کنند و فرزندان `button_frame` از `grid`. چون والدها متفاوت‌اند، مشکلی وجود ندارد.

## ۹. پروژه کلاسی: فرم محاسبه هزینه سفارش

### هدف پروژه

ساخت یک فرم ساده که نام محصول، تعداد و قیمت واحد را دریافت کند و هزینه نهایی را محاسبه کند. پروژه از `Entry`، `Label`، `Button`، تابع، `grid` و مدیریت خطا استفاده می‌کند.

```python
import tkinter as tk
from tkinter import messagebox

root = tk.Tk()
root.title("Order Cost Calculator")
root.geometry("420x280")
root.resizable(False, False)

tk.Label(root, text="Product name:").grid(row=0, column=0, padx=12, pady=8, sticky="w")
product_entry = tk.Entry(root)
product_entry.grid(row=0, column=1, padx=12, pady=8, sticky="ew")

tk.Label(root, text="Quantity:").grid(row=1, column=0, padx=12, pady=8, sticky="w")
quantity_entry = tk.Entry(root)
quantity_entry.grid(row=1, column=1, padx=12, pady=8, sticky="ew")

tk.Label(root, text="Unit price:").grid(row=2, column=0, padx=12, pady=8, sticky="w")
price_entry = tk.Entry(root)
price_entry.grid(row=2, column=1, padx=12, pady=8, sticky="ew")

result_label = tk.Label(root, text="Total: -", font=("Arial", 12, "bold"))
result_label.grid(row=3, column=0, columnspan=2, pady=12)

def calculate_total():
    product_name = product_entry.get().strip()
    quantity_text = quantity_entry.get().strip()
    price_text = price_entry.get().strip()

    if not product_name:
        messagebox.showwarning("Missing Input", "Please enter a product name.")
        return

    try:
        quantity = int(quantity_text)
        unit_price = float(price_text)

        if quantity <= 0:
            messagebox.showwarning("Invalid Quantity", "Quantity must be greater than zero.")
            return

        if unit_price < 0:
            messagebox.showwarning("Invalid Price", "Unit price cannot be negative.")
            return

        total = quantity * unit_price
        result_label.config(text=f"{product_name} | Total: {total:,.2f}")

    except ValueError:
        messagebox.showerror("Invalid Input", "Enter a whole number for quantity and a valid price.")

def clear_form():
    product_entry.delete(0, tk.END)
    quantity_entry.delete(0, tk.END)
    price_entry.delete(0, tk.END)
    result_label.config(text="Total: -")
    product_entry.focus_set()

tk.Button(root, text="Calculate", command=calculate_total).grid(row=4, column=0, padx=12, pady=10, sticky="ew")
tk.Button(root, text="Clear", command=clear_form).grid(row=4, column=1, padx=12, pady=10, sticky="ew")

root.columnconfigure(1, weight=1)
root.mainloop()
```

### تحلیل پروژه

1. سه ورودی برای نام محصول، تعداد و قیمت واحد ساخته شده است.
2. تابع `calculate_total()` داده‌ها را دریافت می‌کند.
3. نام محصول بررسی می‌شود تا خالی نباشد.
4. تعداد با `int()` و قیمت با `float()` تبدیل می‌شوند.
5. تعداد و قیمت از نظر منطقی اعتبارسنجی می‌شوند.
6. نتیجه با `config()` در برچسب نمایش داده می‌شود.
7. تابع `clear_form()` ورودی‌ها و نتیجه را پاک می‌کند.
8. `focus_set()` مکان‌نما را به ورودی نام محصول برمی‌گرداند.

## ۱۰. خطاهای رایج

### خطای ۱: استفاده از پرانتز در `command`

```python
button = tk.Button(root, text="Calculate", command=calculate_total())
```

معمولاً باید این‌گونه باشد:

```python
button = tk.Button(root, text="Calculate", command=calculate_total)
```

### خطای ۲: تلاش برای محاسبه مستقیم با متن `Entry`

```python
result = price_entry.get() * 2
```

اگر هدف محاسبه عددی است، ابتدا ورودی را تبدیل کنید:

```python
price = float(price_entry.get())
result = price * 2
```

در برنامه واقعی، تبدیل باید با مدیریت خطا همراه باشد.

### خطای ۳: استفاده از `pack` و `grid` برای فرزندان یک والد

برای فرزندان یک والد، یک مدیر چیدمان انتخاب کنید.

### خطای ۴: فراموش کردن `mainloop()`

بدون `mainloop()` معمولاً پنجره وارد چرخه عادی دریافت رویدادها نمی‌شود.

### خطای ۵: نام‌گذاری متغیر با نام ویجت یا ماژول

از نام‌های واضح مانند `name_entry`، `result_label` و `calculate_button` استفاده کنید.

## ۱۱. تمرین‌های دانشجو

### تمرین ۱ — معکوس کردن متن

یک ورودی و دو دکمه بسازید:

- دکمه `Reverse` متن را برعکس کند.
- دکمه `Clear` ورودی و نتیجه را پاک کند.

### تمرین ۲ — تبدیل دقیقه به ساعت و دقیقه

کاربر تعداد دقیقه را وارد کند؛ برنامه آن را به ساعت و دقیقه تبدیل کند. برای مثال، ۱۳۵ دقیقه باید به صورت ۲ ساعت و ۱۵ دقیقه نمایش داده شود.

### تمرین ۳ — محاسبه میانگین

سه نمره از کاربر دریافت کنید و میانگین را نمایش دهید. ورودی نامعتبر را مدیریت کنید و اجازه ندهید نمره‌ها خارج از بازه ۰ تا ۲۰ باشند.

### تمرین ۴ — فرم ورود ساده

دو ورودی برای نام کاربری و رمز عبور ایجاد کنید. برای ورودی رمز عبور از `show="*"` استفاده کنید. اگر هر کدام خالی بود، هشدار نمایش دهید؛ در غیر این صورت پیام تأیید نشان دهید. این تمرین صرفاً برای یادگیری رابط کاربری است و احراز هویت واقعی پیاده‌سازی نمی‌کند.

---

## ۱۲. پاسخ تمرین‌ها

### پاسخ تمرین ۱

```python
import tkinter as tk

root = tk.Tk()
root.title("Reverse Text")
root.geometry("350x180")

entry = tk.Entry(root, width=30)
entry.pack(pady=10)

result_label = tk.Label(root, text="")
result_label.pack(pady=5)

def reverse_text():
    result_label.config(text=entry.get()[::-1])

def clear_text():
    entry.delete(0, tk.END)
    result_label.config(text="")

tk.Button(root, text="Reverse", command=reverse_text).pack(pady=3)
tk.Button(root, text="Clear", command=clear_text).pack(pady=3)

root.mainloop()
```

### پاسخ تمرین ۲

```python
import tkinter as tk
from tkinter import messagebox

root = tk.Tk()
root.title("Minutes Converter")
root.geometry("350x180")

entry = tk.Entry(root)
entry.pack(pady=10)

result_label = tk.Label(root, text="")
result_label.pack(pady=5)

def convert_minutes():
    try:
        total_minutes = int(entry.get().strip())

        if total_minutes < 0:
            messagebox.showwarning("Invalid Input", "Minutes cannot be negative.")
            return

        hours = total_minutes // 60
        minutes = total_minutes % 60
        result_label.config(text=f"{hours} hour(s), {minutes} minute(s)")

    except ValueError:
        messagebox.showerror("Invalid Input", "Enter a whole number.")

tk.Button(root, text="Convert", command=convert_minutes).pack(pady=5)

root.mainloop()
```

### پاسخ تمرین ۳

```python
import tkinter as tk
from tkinter import messagebox

root = tk.Tk()
root.title("Grade Average")
root.geometry("380x260")

entries = []

for row in range(3):
    tk.Label(root, text=f"Grade {row + 1}:").grid(row=row, column=0, padx=10, pady=7, sticky="w")
    grade_entry = tk.Entry(root)
    grade_entry.grid(row=row, column=1, padx=10, pady=7, sticky="ew")
    entries.append(grade_entry)

result_label = tk.Label(root, text="")
result_label.grid(row=4, column=0, columnspan=2, pady=8)

def calculate_average():
    try:
        grades = [float(entry.get().strip()) for entry in entries]

        if any(grade < 0 or grade > 20 for grade in grades):
            messagebox.showwarning("Invalid Grade", "Grades must be between 0 and 20.")
            return

        average = sum(grades) / len(grades)
        result_label.config(text=f"Average: {average:.2f}")

    except ValueError:
        messagebox.showerror("Invalid Input", "Enter a valid number for each grade.")

tk.Button(root, text="Calculate Average", command=calculate_average).grid(
    row=3, column=0, columnspan=2, pady=8
)

root.columnconfigure(1, weight=1)
root.mainloop()
```

### پاسخ تمرین ۴

```python
import tkinter as tk
from tkinter import messagebox

root = tk.Tk()
root.title("Login Form Demo")
root.geometry("350x190")

tk.Label(root, text="Username:").grid(row=0, column=0, padx=10, pady=8, sticky="w")
username_entry = tk.Entry(root)
username_entry.grid(row=0, column=1, padx=10, pady=8)

tk.Label(root, text="Password:").grid(row=1, column=0, padx=10, pady=8, sticky="w")
password_entry = tk.Entry(root, show="*")
password_entry.grid(row=1, column=1, padx=10, pady=8)

def check_form():
    username = username_entry.get().strip()
    password = password_entry.get()

    if not username or not password:
        messagebox.showwarning("Missing Input", "Enter both username and password.")
        return

    messagebox.showinfo("Form Submitted", "The form is complete.")

tk.Button(root, text="Submit", command=check_form).grid(
    row=2, column=0, columnspan=2, pady=12
)

root.mainloop()
```

## ۱۳. جمع‌بندی جلسه

در پایان این جلسه، دانشجو باید بتواند:

- [ ] متن `Entry` را با `get()` بخواند.
- [ ] متن ورودی را با `insert()` و `delete()` تغییر دهد.
- [ ] تابع را با `command` به دکمه متصل کند.
- [ ] خطاهای تبدیل داده را با `try/except` مدیریت کند.
- [ ] برای فرم‌های ساده از `grid()` استفاده کند.
- [ ] فاصله‌گذاری و کشیدگی ویجت‌ها را با پارامترهای `pack()` و `grid()` کنترل کند.
- [ ] تفاوت `fill` و `expand` را توضیح دهد.
- [ ] دلیل ترکیب نکردن `pack` و `grid` در یک والد را بیان کند.

## ۱۴. آمادگی برای جلسه بعد

در جلسه بعد، کار با متغیرهای ویژه Tkinter مانند `StringVar` و `IntVar`، گزینه‌های انتخابی مانند `Checkbutton` و `Radiobutton` و پیام‌های استاندارد `messagebox` را یاد می‌گیریم.

پیوند پیشنهادی در Obsidian: `[[Tkinter - جلسه 03 - متغیرها و گزینه‌های انتخابی]]`
