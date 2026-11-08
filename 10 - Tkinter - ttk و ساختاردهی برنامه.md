---
title: "Tkinter - جلسه ۱۰ - طراحی بهتر با ttk و ساختاردهی برنامه"
aliases:
  - Tkinter Session 10
  - ttk و معماری برنامه
tags:
  - Python
  - Tkinter
  - ttk
  - آموزش
level: intermediate
status: ready
---

# جلسه ۱۰ — طراحی بهتر با `ttk` و ساختاردهی برنامه

## هدف جلسه

در جلسات قبل، برنامه To-Do را با امکاناتی مانند افزودن، حذف، جست‌وجو، فیلتر، مرتب‌سازی و ذخیره JSON توسعه دادیم. حالا باید کد را خواناتر و رابط کاربری را منظم‌تر کنیم.

در این جلسه روی چهار مهارت تمرکز می‌کنیم:

1. آشنایی با ویجت‌های `ttk` و تفاوت آن‌ها با ویجت‌های معمولی `tkinter`
2. گروه‌بندی عناصر با `Frame` و چیدمان منظم‌تر
3. جدا کردن منطق برنامه از ساخت رابط کاربری
4. انتقال تدریجی از یک فایل شلوغ به ساختار قابل نگهداری

> نکته مدرس: هدف این جلسه حفظ پروژه جلسه ۹ و بهتر کردن ساختار آن است، نه بازنویسی تمام برنامه از ابتدا.

## ۱. تفاوت `tkinter` و `ttk`

هر دو بخشی از کتابخانه استاندارد رابط گرافیکی پایتون هستند.

- `tkinter` ویجت‌های پایه مانند `Tk`، `Frame`، `Canvas` و `Text` را در اختیار می‌گذارد.
- `tkinter.ttk` مجموعه‌ای از ویجت‌های themed دارد که معمولاً ظاهر هماهنگ‌تری با سیستم و تم انتخاب‌شده دارند.

ویجت‌های رایج `ttk`:

| ویجت | کاربرد |
|---|---|
| `ttk.Button` | دکمه |
| `ttk.Label` | برچسب |
| `ttk.Entry` | ورودی متن |
| `ttk.Frame` | گروه‌بندی عناصر |
| `ttk.LabelFrame` | گروه‌بندی همراه عنوان |
| `ttk.Combobox` | فهرست انتخابی |
| `ttk.Checkbutton` | گزینه روشن/خاموش |
| `ttk.Radiobutton` | انتخاب یک گزینه از چند گزینه |
| `ttk.Treeview` | نمایش داده‌ها در قالب ردیف و ستون |
| `ttk.Progressbar` | نمایش پیشرفت عملیات |
| `ttk.Separator` | خط جداکننده |

نکته: بعضی ویجت‌های پایه مانند `Text` و `Listbox` معادل مستقیم و یکسانی در `ttk` ندارند؛ بنابراین استفاده ترکیبی از `tkinter` و `ttk` طبیعی است.

## ۲. یک مثال ساده از `ttk`

```python
import tkinter as tk
from tkinter import ttk

root = tk.Tk()
root.title("ttk Example")
root.geometry("420x220")

main_frame = ttk.Frame(root, padding=16)
main_frame.pack(fill="both", expand=True)

title_label = ttk.Label(
    main_frame,
    text="Student Information",
    font=("Arial", 14, "bold")
)
title_label.pack(anchor="w", pady=(0, 12))

name_entry = ttk.Entry(main_frame)
name_entry.pack(fill="x", pady=5)

save_button = ttk.Button(
    main_frame,
    text="Save",
    command=lambda: print(name_entry.get())
)
save_button.pack(anchor="e", pady=8)

root.mainloop()
```

### نکته‌های آموزشی

- `padding=16` فضای داخلی قاب را تنظیم می‌کند.
- `fill="x"` اجازه می‌دهد ورودی در عرض قاب کشیده شود.
- `anchor="w"` عنصر را به سمت چپ قاب می‌برد.
- `pady=(0, 12)` فاصله عمودی متفاوتی در بالا و پایین ایجاد می‌کند.

## ۳. استفاده از `ttk.Style`

`ttk.Style` برای تنظیم برخی ویژگی‌های ظاهری ویجت‌های themed به کار می‌رود.

```python
import tkinter as tk
from tkinter import ttk

root = tk.Tk()
root.title("Style Example")

style = ttk.Style(root)
style.configure(
    "Title.TLabel",
    font=("Arial", 16, "bold")
)
style.configure(
    "Accent.TButton",
    padding=(12, 6)
)

title = ttk.Label(
    root,
    text="Task Manager",
    style="Title.TLabel"
)
title.pack(padx=16, pady=16)

button = ttk.Button(
    root,
    text="Continue",
    style="Accent.TButton"
)
button.pack(pady=8)

root.mainloop()
```

نکته‌های مهم:

- نام سبک یک قرارداد نام‌گذاری است؛ مثلاً `Title.TLabel` به سبک یک `TLabel` اشاره می‌کند.
- پشتیبانی از رنگ‌ها و برخی ویژگی‌های ظاهری به تم و سیستم‌عامل وابسته است.
- برای تغییر همه جزئیات ظاهری `ttk` نباید فرض کنیم همه گزینه‌های ویجت‌های معمولی `tkinter` در دسترس‌اند.

## ۴. گروه‌بندی منطقی رابط کاربری

در برنامه‌های بزرگ، قرار دادن تمام ویجت‌ها مستقیماً در پنجره اصلی باعث شلوغی کد می‌شود. بهتر است رابط را به بخش‌های معنی‌دار تقسیم کنیم:

- بخش افزودن کار
- بخش جست‌وجو و فیلتر
- بخش فهرست
- بخش عملیات
- نوار وضعیت

هر بخش می‌تواند یک `Frame` جداگانه داشته باشد.

نمونه:

```python
import tkinter as tk
from tkinter import ttk

root = tk.Tk()
root.title("Grouped Layout")
root.geometry("520x320")

main_frame = ttk.Frame(root, padding=12)
main_frame.pack(fill="both", expand=True)

input_frame = ttk.LabelFrame(
    main_frame,
    text="New Task",
    padding=10
)
input_frame.pack(fill="x", pady=(0, 10))

task_entry = ttk.Entry(input_frame)
task_entry.pack(side="left", fill="x", expand=True, padx=(0, 8))

ttk.Button(
    input_frame,
    text="Add",
    command=lambda: print(task_entry.get())
).pack(side="left")

list_frame = ttk.LabelFrame(
    main_frame,
    text="Tasks",
    padding=10
)
list_frame.pack(fill="both", expand=True)

task_list = tk.Listbox(list_frame, height=8)
task_list.pack(fill="both", expand=True)

root.mainloop()
```

## ۵. قانون مهم چیدمان

در یک والد مشخص، بهتر است `pack` و `grid` را با هم ترکیب نکنیم.

این نمونه نادرست است:

```python
# Avoid mixing pack and grid in the same parent.
button_a.pack()
button_b.grid(row=0, column=0)
```

اگر دو ویجت در یک والد باشند، انتخاب یک مدیر چیدمان برای همان والد، رفتار قابل پیش‌بینی‌تری ایجاد می‌کند. البته می‌توان در `Frame`های جدا از مدیرهای متفاوت استفاده کرد؛ مثلاً در پنجره اصلی `pack` و داخل یک قاب فرزند `grid`.

## ۶. جداسازی رابط کاربری از منطق برنامه

یکی از عادت‌های مهم در توسعه نرم‌افزار این است که هر تابع یک مسئولیت روشن داشته باشد.

در برنامه To-Do، مسئولیت‌ها می‌توانند این‌طور تقسیم شوند:

- `load_tasks()`: خواندن داده‌ها
- `save_tasks()`: ذخیره داده‌ها
- `add_task()`: افزودن یک کار
- `delete_task()`: حذف کار انتخاب‌شده
- `refresh_list()`: به‌روزرسانی نمایش
- `_build_ui()`: ساخت ویجت‌ها و چیدمان

پیشوند زیرخط در نام `_build_ui()` نشان می‌دهد این تابع در طراحی کلاس، یک متد داخلی محسوب می‌شود؛ این نام‌گذاری به‌تنهایی دسترسی را از بیرون مسدود نمی‌کند.

برای پروژه‌های بزرگ‌تر می‌توانیم منطق داده را در یک کلاس یا ماژول جدا قرار دهیم. در این جلسه فقط مسئولیت‌ها را شفاف‌تر می‌کنیم تا پروژه بیش از حد پیچیده نشود.

## ۷. پروژه عملی: بازآرایی رابط To-Do

این نمونه کوچک، الگوی ساختاری را نشان می‌دهد. هدف آن نمایش قاب‌بندی با `ttk` است؛ قابلیت‌های کامل ذخیره JSON، جست‌وجو و فیلتر همچنان می‌توانند از پروژه جلسه ۹ حفظ شوند.

```python
import tkinter as tk
from tkinter import ttk


class TodoLayout:
    def __init__(self, root):
        self.root = root
        self.root.title("Task Manager")
        self.root.geometry("560x420")

        self.tasks = []

        self._build_ui()

    def _build_ui(self):
        main_frame = ttk.Frame(self.root, padding=12)
        main_frame.pack(fill="both", expand=True)

        input_frame = ttk.LabelFrame(
            main_frame,
            text="New Task",
            padding=10
        )
        input_frame.pack(fill="x", pady=(0, 10))

        self.task_entry = ttk.Entry(input_frame)
        self.task_entry.pack(
            side="left",
            fill="x",
            expand=True,
            padx=(0, 8)
        )
        self.task_entry.bind("<Return>", self.add_task)

        ttk.Button(
            input_frame,
            text="Add",
            command=self.add_task
        ).pack(side="left")

        list_frame = ttk.LabelFrame(
            main_frame,
            text="Task List",
            padding=10
        )
        list_frame.pack(fill="both", expand=True)

        self.task_list = tk.Listbox(list_frame, height=10)
        self.task_list.pack(fill="both", expand=True)

        action_frame = ttk.Frame(main_frame)
        action_frame.pack(fill="x", pady=(10, 0))

        ttk.Button(
            action_frame,
            text="Delete Selected",
            command=self.delete_selected
        ).pack(side="left")

        self.status_label = ttk.Label(
            action_frame,
            text="0 tasks"
        )
        self.status_label.pack(side="right")

    def add_task(self, event=None):
        title = self.task_entry.get().strip()

        if not title:
            return

        self.tasks.append(title)
        self.task_entry.delete(0, tk.END)
        self.refresh_list()

    def delete_selected(self):
        selection = self.task_list.curselection()

        if not selection:
            return

        del self.tasks[selection[0]]
        self.refresh_list()

    def refresh_list(self):
        self.task_list.delete(0, tk.END)

        for title in self.tasks:
            self.task_list.insert(tk.END, title)

        self.status_label.config(text=f"{len(self.tasks)} tasks")


if __name__ == "__main__":
    root = tk.Tk()
    app = TodoLayout(root)
    root.mainloop()
```

### توسعه کلاسی پروژه

پس از اجرای نمونه، از دانشجو بخواهید:

1. کادر جست‌وجو و فیلتر جلسه ۹ را اضافه کند.
2. داده‌ها را به فایل JSON متصل کند.
3. برای ورودی و فهرست، قاب‌های جداگانه نگه دارد.
4. تابع‌های رابط کاربری و ذخیره‌سازی را از هم تشخیص دهد.

## ۸. تمرین‌های دانشجو

### تمرین ۱ — تبدیل دکمه‌ها به `ttk`

در یک برنامه ساده، `tk.Button` و `tk.Label` را با `ttk.Button` و `ttk.Label` جایگزین کنید.

**پاسخ نمونه:**

```python
import tkinter as tk
from tkinter import ttk

root = tk.Tk()

ttk.Label(root, text="Hello").pack(padx=10, pady=10)
ttk.Button(root, text="Close", command=root.destroy).pack(pady=5)

root.mainloop()
```

### تمرین ۲ — قاب‌بندی فرم

دو ورودی عنوان و توضیحات را در یک `LabelFrame` قرار دهید و دکمه ذخیره را پایین آن بگذارید.

**پاسخ پیشنهادی:**

```python
import tkinter as tk
from tkinter import ttk

root = tk.Tk()
root.geometry("420x260")

form = ttk.LabelFrame(root, text="Task Details", padding=12)
form.pack(fill="x", padx=12, pady=12)

ttk.Label(form, text="Title").grid(row=0, column=0, sticky="w", pady=5)
ttk.Entry(form).grid(row=0, column=1, sticky="ew", pady=5)

ttk.Label(form, text="Description").grid(row=1, column=0, sticky="w", pady=5)
ttk.Entry(form).grid(row=1, column=1, sticky="ew", pady=5)

form.columnconfigure(1, weight=1)

ttk.Button(root, text="Save").pack(anchor="e", padx=12)

root.mainloop()
```

### تمرین ۳ — نمایش وضعیت

در برنامه To-Do تعداد کارها را در یک `ttk.Label` نمایش دهید.

**پاسخ:**

```python
self.status_label.config(text=f"{len(self.tasks)} tasks")
```

این خط را پس از به‌روزرسانی فهرست اجرا کنید.

### تمرین ۴ — توضیح معماری

توضیح دهید چرا بهتر است کد ساخت رابط کاربری در `_build_ui()` قرار بگیرد و کد ذخیره فایل در تابع جداگانه‌ای مانند `save_tasks()` باشد.

**پاسخ مورد انتظار:** این جداسازی مسئولیت‌ها را روشن می‌کند، تکرار کد را کاهش می‌دهد، تست و عیب‌یابی را آسان‌تر می‌کند و تغییر ظاهر برنامه را بدون دستکاری منطق ذخیره‌سازی ممکن می‌سازد.

## ۹. خطاهای رایج

- **فرض اینکه همه ویجت‌ها معادل `ttk` دارند:** برای مثال، `Text` و `Listbox` معمولاً از `tkinter` استفاده می‌شوند.
- **ترکیب `pack` و `grid` در یک والد:** هر والد باید چیدمان قابل پیش‌بینی داشته باشد.
- **قرار دادن تمام کد در یک تابع:** با افزایش امکانات، نگهداری و عیب‌یابی سخت می‌شود.
- **استفاده از `ttk.Style` با فرض یکسان بودن همه تم‌ها:** ظاهر نهایی می‌تواند به سیستم‌عامل و تم وابسته باشد.
- **حذف امکانات قبلی هنگام بازآرایی:** در پروژه اصلی، ذخیره JSON، جست‌وجو و فیلتر را حفظ کنید؛ نمونه این جلسه فقط الگوی رابط را ساده کرده است.

## ۱۰. جمع‌بندی جلسه

- `ttk` ویجت‌هایی با ظاهر themed فراهم می‌کند.
- `ttk.Style` برای تنظیم برخی ویژگی‌های ظاهری به کار می‌رود.
- `Frame` و `LabelFrame` به گروه‌بندی منطقی عناصر کمک می‌کنند.
- `pack` و `grid` را در یک والد با هم ترکیب نکنید.
- جدا کردن ساخت رابط، منطق برنامه و ذخیره‌سازی، توسعه پروژه را آسان‌تر می‌کند.

## ۱۱. چک‌لیست مدرس

- [ ] تفاوت `tkinter` و `ttk` را توضیح دادم.
- [ ] یک `ttk.Style` ساده تعریف کردم.
- [ ] رابط را با چند `Frame` گروه‌بندی کردم.
- [ ] قانون استفاده از `pack` و `grid` را تمرین کردیم.
- [ ] دانشجو تابع ساخت رابط و تابع عملیات را از هم تشخیص داد.
- [ ] امکانات قبلی پروژه To-Do را در برنامه اصلی حفظ کردیم.

## جلسه بعد

در جلسه ۱۱ وارد `ttk.Treeview` می‌شویم؛ ویجتی برای نمایش اطلاعات در قالب جدول. این ابزار مقدمه مناسبی برای پروژه‌های مدیریتی و اتصال آینده به SQLite است.
