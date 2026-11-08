---
title: "Tkinter - جلسه 03 - متغیرها و گزینه‌های انتخابی"
aliases:
  - "Tkinter Session 3"
  - "متغیرهای Tkinter و کنترل‌های انتخابی"
tags:
  - Python
  - Tkinter
  - GUI
  - Teaching
level: Beginner
status: Ready
language: Persian
session: 3
---

# جلسه ۳ — متغیرهای Tkinter و گزینه‌های انتخابی

> [!abstract] هدف جلسه
> در این جلسه یاد می‌گیریم مقدار ویجت‌ها را با متغیرهای ویژه Tkinter مدیریت کنیم، گزینه‌های انتخابی بسازیم، از پیام‌های استاندارد استفاده کنیم و یک فرم سفارش ساده طراحی کنیم.

## ۱. پیش‌نیازهای جلسه

دانشجو بهتر است بتواند:

- پنجره با `Tk()` بسازد.
- از `Label`، `Entry` و `Button` استفاده کند.
- با `get()` و `delete()` مقدار ورودی را بخواند یا پاک کند.
- تابع بنویسد و آن را به `command` متصل کند.
- ورودی‌ها را با `try/except` بررسی کند.
- از `grid()` و `pack()` برای چیدمان استفاده کند.

## ۲. متغیرهای معمولی در برابر متغیرهای Tkinter

در پایتون از متغیرهای معمولی استفاده می‌کنیم:

```python
name = "Sara"
age = 20
price = 12.5
is_active = True
```

Tkinter متغیرهای ویژه‌ای دارد که می‌توان آن‌ها را به بعضی ویجت‌ها وصل کرد. وقتی مقدار این متغیرها را تغییر دهیم، ویجت متصل می‌تواند مقدار جدید را نمایش دهد.

چهار نوع پرکاربرد:

| نوع متغیر    | کاربرد معمول      | نمونه مقدار |
| ------------ | ----------------- | ----------- |
| `StringVar`  | متن               | `"Sara"`    |
| `IntVar`     | عدد صحیح          | `3`         |
| `DoubleVar`  | عدد اعشاری        | `12.5`      |
| `BooleanVar` | مقدار درست/نادرست | `True`      |

نمونه:

```python
import tkinter as tk

root = tk.Tk()
root.title("Tkinter Variables")
root.geometry("350x180")

name_var = tk.StringVar(value="Student")

name_entry = tk.Entry(root, textvariable=name_var)
name_entry.pack(pady=8)

result_label = tk.Label(root, textvariable=name_var)
result_label.pack(pady=8)

def show_value():
    print(name_var.get())

tk.Button(root, text="Print Value", command=show_value).pack(pady=5)

root.mainloop()
```

در این مثال، `Entry` و `Label` هر دو به یک `StringVar` وصل شده‌اند؛ بنابراین مقدارشان از یک منبع مشترک می‌آید.

### متدهای اصلی متغیرها

- `get()` مقدار فعلی متغیر را برمی‌گرداند.
- `set(value)` مقدار متغیر را تغییر می‌دهد.

```python
name_var.set("Alex")
print(name_var.get())
```

## ۳. استفاده از `StringVar` برای نمایش نتیجه

```python
import tkinter as tk

root = tk.Tk()
root.title("StringVar Result")
root.geometry("350x180")

name_var = tk.StringVar()
result_var = tk.StringVar(value="Your result will appear here")

tk.Label(root, text="Name:").pack(pady=5)
tk.Entry(root, textvariable=name_var).pack(pady=5)

tk.Label(root, textvariable=result_var).pack(pady=8)

def greet():
    name = name_var.get().strip()

    if not name:
        result_var.set("Please enter your name.")
        return

    result_var.set(f"Hello, {name}!")

tk.Button(root, text="Greet", command=greet).pack(pady=5)

root.mainloop()
```

در این مثال برای نمایش نتیجه از `result_var.set()` استفاده کرده‌ایم؛ بنابراین نیازی به صدا زدن `result_label.config()` نیست.

## ۴. ویجت `Checkbutton`

از `Checkbutton` برای انتخاب مستقل یک گزینه استفاده می‌کنیم؛ مثلاً «افزودن پنیر» یا «قبول قوانین».

```python
import tkinter as tk

root = tk.Tk()
root.title("Checkbutton Example")
root.geometry("350x180")

cheese_var = tk.BooleanVar(value=False)

cheese_check = tk.Checkbutton(
    root,
    text="Add extra cheese",
    variable=cheese_var
)
cheese_check.pack(pady=10)

def show_selection():
    if cheese_var.get():
        print("Extra cheese selected")
    else:
        print("Extra cheese not selected")

tk.Button(root, text="Show Selection", command=show_selection).pack(pady=5)

root.mainloop()
```

نکته‌ها:

- `variable=cheese_var` متغیر را به گزینه متصل می‌کند.
- وقتی گزینه انتخاب شود، مقدار `cheese_var.get()` معمولاً `True` است.
- با برداشتن انتخاب، مقدار آن `False` می‌شود.
- برای چند انتخاب مستقل، معمولاً برای هر گزینه یک متغیر جدا تعریف می‌کنیم.

## ۵. ویجت `Radiobutton`

از `Radiobutton` برای انتخاب یکی از چند گزینه هم‌گروه استفاده می‌کنیم؛ مانند انتخاب اندازه کوچک، متوسط یا بزرگ.

```python
import tkinter as tk

root = tk.Tk()
root.title("Radiobutton Example")
root.geometry("350x220")

size_var = tk.StringVar(value="Medium")

tk.Label(root, text="Choose a size:").pack(pady=5)

tk.Radiobutton(
    root,
    text="Small",
    variable=size_var,
    value="Small"
).pack(anchor="w", padx=25)

tk.Radiobutton(
    root,
    text="Medium",
    variable=size_var,
    value="Medium"
).pack(anchor="w", padx=25)

tk.Radiobutton(
    root,
    text="Large",
    variable=size_var,
    value="Large"
).pack(anchor="w", padx=25)

def show_size():
    print(size_var.get())

tk.Button(root, text="Show Size", command=show_size).pack(pady=8)

root.mainloop()
```

نکته کلیدی: تمام گزینه‌های یک گروه باید به یک متغیر مشترک وصل شوند، اما `value` هر گزینه متفاوت باشد. انتخاب یک گزینه، مقدار متغیر را به همان `value` تغییر می‌دهد.

## ۶. تفاوت `Checkbutton` و `Radiobutton`

|    ویژگی     |             `Checkbutton`              |          `Radiobutton`          |
| :----------: | :------------------------------------: | :-----------------------------: |
|    کاربرد    |              انتخاب مستقل              |     انتخاب یکی از چند گزینه     |
| تعداد انتخاب | چند گزینه می‌توانند هم‌زمان فعال باشند | در یک گروه معمولاً فقط یک گزینه |
|  متغیر رایج  |              `BooleanVar`              |     `StringVar` یا `IntVar`     |
|     مثال     |            افزودنی‌های غذا             |           اندازه غذا            |

## ۷. پیام‌های استاندارد با `messagebox`

ماژول `messagebox` پنجره‌های پیام آماده فراهم می‌کند.

```python
from tkinter import messagebox
```

توابع رایج:

| تابع | کاربرد |
|---|---|
| `showinfo(title, message)` | پیام اطلاع‌رسانی |
| `showwarning(title, message)` | هشدار |
| `showerror(title, message)` | خطا |
| `askyesno(title, message)` | پرسش با پاسخ بولی |
| `askokcancel(title, message)` | تأیید یا لغو |
| `askretrycancel(title, message)` | تلاش مجدد یا لغو |

نمونه:

```python
import tkinter as tk
from tkinter import messagebox

root = tk.Tk()
root.title("Message Box")
root.geometry("300x140")

def confirm_action():
    answer = messagebox.askyesno(
        "Confirm",
        "Do you want to continue?"
    )

    if answer:
        messagebox.showinfo("Confirmed", "You chose Yes.")
    else:
        messagebox.showwarning("Cancelled", "The action was cancelled.")

tk.Button(root, text="Continue", command=confirm_action).pack(pady=30)

root.mainloop()
```

`askyesno()` مقدار `True` یا `False` برمی‌گرداند؛ بنابراین می‌توان نتیجه را با `if` بررسی کرد.

## ۸. پروژه کلاسی: فرم سفارش پیتزا

### امکانات پروژه

- دریافت نام مشتری
- انتخاب اندازه پیتزا با `Radiobutton`
- انتخاب افزودنی‌ها با `Checkbutton`
- محاسبه هزینه سفارش
- نمایش نتیجه با `StringVar`
- اعتبارسنجی نام
- نمایش خلاصه سفارش با `messagebox`
- پاک کردن فرم با دکمه `Reset`

### قوانین قیمت‌گذاری نمونه

| گزینه | قیمت نمونه |
|---|---:|
| کوچک | 150 |
| متوسط | 220 |
| بزرگ | 300 |
| پنیر اضافه | 30 |
| قارچ | 25 |
| زیتون | 20 |

واحد قیمت در این تمرین دلخواه است؛ در صورت نیاز آن را با واحد پول موردنظر کلاس جایگزین کنید.

```python
import tkinter as tk
from tkinter import messagebox

root = tk.Tk()
root.title("Pizza Order Form")
root.geometry("440x520")
root.resizable(False, False)

name_var = tk.StringVar()
size_var = tk.StringVar(value="Medium")
cheese_var = tk.BooleanVar(value=False)
mushroom_var = tk.BooleanVar(value=False)
olive_var = tk.BooleanVar(value=False)
result_var = tk.StringVar(value="Complete the form and calculate your order.")

tk.Label(root, text="Pizza Order Form", font=("Arial", 16, "bold")).pack(pady=10)

customer_frame = tk.LabelFrame(root, text="Customer")
customer_frame.pack(fill="x", padx=15, pady=7)

tk.Label(customer_frame, text="Name:").grid(row=0, column=0, padx=8, pady=8, sticky="w")
tk.Entry(customer_frame, textvariable=name_var).grid(
    row=0, column=1, padx=8, pady=8, sticky="ew"
)
customer_frame.columnconfigure(1, weight=1)

size_frame = tk.LabelFrame(root, text="Pizza Size")
size_frame.pack(fill="x", padx=15, pady=7)

for size in ("Small", "Medium", "Large"):
    tk.Radiobutton(
        size_frame,
        text=size,
        variable=size_var,
        value=size
    ).pack(anchor="w", padx=10)

topping_frame = tk.LabelFrame(root, text="Extra Toppings")
topping_frame.pack(fill="x", padx=15, pady=7)

tk.Checkbutton(
    topping_frame,
    text="Extra cheese (+30)",
    variable=cheese_var
).pack(anchor="w", padx=10)

tk.Checkbutton(
    topping_frame,
    text="Mushrooms (+25)",
    variable=mushroom_var
).pack(anchor="w", padx=10)

tk.Checkbutton(
    topping_frame,
    text="Olives (+20)",
    variable=olive_var
).pack(anchor="w", padx=10)

result_label = tk.Label(
    root,
    textvariable=result_var,
    wraplength=390,
    justify="center"
)
result_label.pack(padx=15, pady=12)

def calculate_order():
    customer_name = name_var.get().strip()

    if not customer_name:
        messagebox.showwarning("Missing Name", "Please enter the customer name.")
        return

    size_prices = {
        "Small": 150,
        "Medium": 220,
        "Large": 300
    }

    total = size_prices[size_var.get()]
    toppings = []

    if cheese_var.get():
        total += 30
        toppings.append("Extra cheese")

    if mushroom_var.get():
        total += 25
        toppings.append("Mushrooms")

    if olive_var.get():
        total += 20
        toppings.append("Olives")

    if not toppings:
        toppings_text = "None"
    else:
        toppings_text = ", ".join(toppings)

    summary = (
        f"Customer: {customer_name}\n"
        f"Size: {size_var.get()}\n"
        f"Toppings: {toppings_text}\n"
        f"Total: {total}"
    )

    result_var.set(f"Order total: {total}")
    messagebox.showinfo("Order Summary", summary)

def reset_form():
    name_var.set("")
    size_var.set("Medium")
    cheese_var.set(False)
    mushroom_var.set(False)
    olive_var.set(False)
    result_var.set("Complete the form and calculate your order.")

button_frame = tk.Frame(root)
button_frame.pack(pady=8)

tk.Button(
    button_frame,
    text="Calculate Order",
    command=calculate_order
).grid(row=0, column=0, padx=6)

tk.Button(
    button_frame,
    text="Reset",
    command=reset_form
).grid(row=0, column=1, padx=6)

root.mainloop()
```

## ۹. تحلیل پروژه مرحله‌به‌مرحله

1. `name_var` نام مشتری را نگه می‌دارد.
2. `size_var` گزینه اندازه انتخاب‌شده را نگه می‌دارد.
3. سه متغیر بولی، وضعیت افزودنی‌ها را ثبت می‌کنند.
4. `LabelFrame` بخش‌های فرم را گروه‌بندی می‌کند.
5. تابع `calculate_order()` ابتدا نام را بررسی می‌کند.
6. قیمت پایه با استفاده از دیکشنری `size_prices` انتخاب می‌شود.
7. قیمت افزودنی‌های انتخاب‌شده به مبلغ پایه اضافه می‌شود.
8. خلاصه سفارش با `messagebox.showinfo()` نمایش داده می‌شود.
9. تابع `reset_form()` متغیرها را به مقادیر اولیه برمی‌گرداند.

## ۱۰. خطاهای رایج

### خطای ۱: تعریف متغیر بدون اتصال آن به ویجت

تعریف `StringVar` به‌تنهایی کافی نیست؛ باید آن را به ویجت وصل کنیم:

```python
name_var = tk.StringVar()
name_entry = tk.Entry(root, textvariable=name_var)
```

### خطای ۲: استفاده از متغیر جدا برای هر `Radiobutton`

اگر هر گزینه متغیر جداگانه داشته باشد، انتخاب انحصاری مورد انتظار به‌درستی مدیریت نمی‌شود. برای گزینه‌های یک گروه از یک متغیر مشترک استفاده کنید.

### خطای ۳: استفاده از `Checkbutton` برای انتخاب انحصاری

اگر کاربر باید دقیقاً یکی از چند گزینه را انتخاب کند، معمولاً `Radiobutton` انتخاب مناسب‌تری است.

### خطای ۴: فراموش کردن `value` در `Radiobutton`

هر گزینه باید مقدار مشخصی داشته باشد تا مقدار انتخاب‌شده در متغیر قابل تشخیص باشد.

### خطای ۵: فراموش کردن `return` بعد از نمایش خطا

اگر ورودی نامعتبر است، پس از نمایش هشدار معمولاً باید با `return` ادامه تابع را متوقف کنیم.

## ۱۱. تمرین‌های دانشجو

### تمرین ۱ — انتخاب روش ارسال

فرمی بسازید که کاربر از بین گزینه‌های `Standard`، `Express` و `Pickup` یکی را انتخاب کند. با کلیک روی دکمه، گزینه انتخاب‌شده در یک `Label` نمایش داده شود.

### تمرین ۲ — افزودنی‌های ساندویچ

سه `Checkbutton` برای پنیر، قارچ و سس بسازید. هر گزینه قیمت جداگانه داشته باشد. با کلیک روی دکمه، مبلغ نهایی نمایش داده شود.

### تمرین ۳ — تأیید خروج

دکمه‌ای با عنوان `Exit` بسازید. با کلیک روی آن، `askyesno()` نمایش داده شود؛ اگر کاربر `Yes` را انتخاب کرد، پنجره بسته شود و اگر `No` را انتخاب کرد، برنامه باز بماند.

### تمرین ۴ — فرم ثبت‌نام ساده

نام، رشته تحصیلی و وضعیت موافقت با قوانین را دریافت کنید. رشته تحصیلی با `Radiobutton` و موافقت با قوانین با `Checkbutton` انتخاب شود. اگر نام خالی یا قوانین تأیید نشده بود، هشدار نمایش دهید؛ در غیر این صورت خلاصه اطلاعات را نمایش دهید.

---

## ۱۲. پاسخ تمرین‌ها

### پاسخ تمرین ۱

```python
import tkinter as tk

root = tk.Tk()
root.title("Delivery Method")
root.geometry("350x220")

delivery_var = tk.StringVar(value="Standard")
result_var = tk.StringVar(value="Choose a delivery method.")

tk.Radiobutton(root, text="Standard", variable=delivery_var, value="Standard").pack(anchor="w", padx=20)
tk.Radiobutton(root, text="Express", variable=delivery_var, value="Express").pack(anchor="w", padx=20)
tk.Radiobutton(root, text="Pickup", variable=delivery_var, value="Pickup").pack(anchor="w", padx=20)

tk.Label(root, textvariable=result_var).pack(pady=8)

def show_delivery():
    result_var.set(f"Selected: {delivery_var.get()}")

tk.Button(root, text="Show Selection", command=show_delivery).pack(pady=5)

root.mainloop()
```

### پاسخ تمرین ۲

```python
import tkinter as tk

root = tk.Tk()
root.title("Sandwich Extras")
root.geometry("350x250")

cheese_var = tk.BooleanVar()
mushroom_var = tk.BooleanVar()
sauce_var = tk.BooleanVar()
result_var = tk.StringVar(value="Total: 0")

tk.Checkbutton(root, text="Cheese (+20)", variable=cheese_var).pack(anchor="w", padx=20)
tk.Checkbutton(root, text="Mushrooms (+15)", variable=mushroom_var).pack(anchor="w", padx=20)
tk.Checkbutton(root, text="Sauce (+10)", variable=sauce_var).pack(anchor="w", padx=20)

tk.Label(root, textvariable=result_var).pack(pady=8)

def calculate_total():
    total = 0

    if cheese_var.get():
        total += 20
    if mushroom_var.get():
        total += 15
    if sauce_var.get():
        total += 10

    result_var.set(f"Total: {total}")

tk.Button(root, text="Calculate", command=calculate_total).pack(pady=5)

root.mainloop()
```

### پاسخ تمرین ۳

```python
import tkinter as tk
from tkinter import messagebox

root = tk.Tk()
root.title("Exit Confirmation")
root.geometry("300x140")

def confirm_exit():
    answer = messagebox.askyesno("Exit", "Do you want to close the application?")

    if answer:
        root.destroy()

tk.Button(root, text="Exit", command=confirm_exit).pack(pady=35)

root.mainloop()
```

### پاسخ تمرین ۴

```python
import tkinter as tk
from tkinter import messagebox

root = tk.Tk()
root.title("Registration Form")
root.geometry("380x300")

name_var = tk.StringVar()
major_var = tk.StringVar(value="Software")
accepted_var = tk.BooleanVar(value=False)

tk.Label(root, text="Name:").grid(row=0, column=0, padx=10, pady=8, sticky="w")
tk.Entry(root, textvariable=name_var).grid(row=0, column=1, padx=10, pady=8)

tk.Label(root, text="Major:").grid(row=1, column=0, padx=10, pady=8, sticky="w")

major_frame = tk.Frame(root)
major_frame.grid(row=1, column=1, sticky="w")

tk.Radiobutton(major_frame, text="Software", variable=major_var, value="Software").pack(anchor="w")
tk.Radiobutton(major_frame, text="Network", variable=major_var, value="Network").pack(anchor="w")
tk.Radiobutton(major_frame, text="IT", variable=major_var, value="IT").pack(anchor="w")

tk.Checkbutton(
    root,
    text="I accept the rules",
    variable=accepted_var
).grid(row=2, column=0, columnspan=2, padx=10, pady=10)

def submit_form():
    name = name_var.get().strip()

    if not name:
        messagebox.showwarning("Missing Name", "Please enter your name.")
        return

    if not accepted_var.get():
        messagebox.showwarning("Rules Required", "Please accept the rules.")
        return

    summary = f"Name: {name}\nMajor: {major_var.get()}\nRules accepted: Yes"
    messagebox.showinfo("Registration Summary", summary)

tk.Button(root, text="Submit", command=submit_form).grid(
    row=3, column=0, columnspan=2, pady=12
)

root.mainloop()
```

## ۱۳. جمع‌بندی و چک‌لیست

در پایان جلسه، دانشجو باید بتواند:

- [ ] تفاوت متغیر معمولی و متغیر ویژه Tkinter را توضیح دهد.
- [ ] از `StringVar`، `IntVar`، `DoubleVar` و `BooleanVar` استفاده کند.
- [ ] مقدار متغیر را با `get()` بخواند و با `set()` تغییر دهد.
- [ ] با `Checkbutton` گزینه‌های مستقل بسازد.
- [ ] با `Radiobutton` انتخاب انحصاری ایجاد کند.
- [ ] از `messagebox` برای اطلاع‌رسانی، هشدار و تأیید استفاده کند.
- [ ] یک فرم چندبخشی ساده بسازد و ورودی‌های آن را اعتبارسنجی کند.

## ۱۴. آمادگی برای جلسه بعد

در جلسه چهارم، ویجت‌های انتخاب و فهرست را بررسی می‌کنیم؛ از جمله `Listbox`، `Combobox` از ماژول `tkinter.ttk` و `Spinbox`. سپس یک فرم انتخاب محصول یا دوره می‌سازیم.

پیوند پیشنهادی در Obsidian: `[[Tkinter - جلسه 04 - Listbox و Combobox]]`
