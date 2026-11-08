---
title: "Tkinter - جلسه ۸ - ذخیره‌سازی JSON و برنامه To-Do"
aliases:
  - Tkinter Session 8
  - JSON و To-Do List
tags:
  - Python
  - Tkinter
  - JSON
  - آموزش
level: intermediate
status: ready
---

# جلسه ۸ — ذخیره‌سازی JSON و ساخت برنامه To-Do List

## هدف جلسه

تا اینجا برنامه‌های Tkinter ساخته‌ایم که هنگام اجرای برنامه کار می‌کنند؛ اما معمولاً با بستن برنامه، اطلاعاتی که کاربر وارد کرده از بین می‌روند. در این جلسه یاد می‌گیریم اطلاعات را در یک فایل JSON ذخیره کنیم و هنگام اجرای دوباره برنامه آن‌ها را بخوانیم.

در پایان، یک برنامه فهرست کارها (To-Do List) خواهیم داشت که می‌تواند کار جدید اضافه کند، کار انتخاب‌شده را حذف کند، وضعیت انجام کار را تغییر دهد و اطلاعات را در فایل نگه دارد.

## پیش‌نیازها

- ساخت پنجره و کار با `Button`، `Entry`، `Label` و `Frame`
- کار با `Listbox`
- تعریف تابع و مدیریت رویدادها
- خواندن و نوشتن فایل متنی
- آشنایی مقدماتی با `try/except`

## ۱. JSON چیست؟

JSON یک قالب متنی برای نگهداری و تبادل داده است. این قالب برای ذخیره اطلاعاتی مانند نام، وضعیت، تنظیمات و فهرست‌ها بسیار مناسب است.

نمونه یک فایل JSON:

```json
{
    "tasks": [
        {
            "title": "Read Python notes",
            "done": false
        },
        {
            "title": "Practice Tkinter",
            "done": true
        }
    ]
}
```

در این مثال:

- `{}` یک شیء JSON را نشان می‌دهد.
- `[]` یک آرایه یا فهرست را نشان می‌دهد.
- `title` نام کار است.
- `done` وضعیت انجام کار را با مقدار `true` یا `false` نگه می‌دارد.

در پایتون، مقادیر JSON به انواع داده متناظر تبدیل می‌شوند:

| JSON | Python |
|---|---|
| Object | `dict` |
| Array | `list` |
| String | `str` |
| Number | `int` یا `float` |
| `true` / `false` | `True` / `False` |
| `null` | `None` |

## ۲. ماژول `json`

ماژول `json` همراه پایتون نصب می‌شود و معمولاً نیازی به نصب جداگانه ندارد.

### تبدیل داده پایتون به رشته JSON

```python
import json

data = {
    "name": "Sara",
    "age": 20,
    "active": True
}

json_text = json.dumps(data, indent=4)
print(json_text)
```

`json.dumps()` داده پایتون را به یک رشته JSON تبدیل می‌کند.

### تبدیل رشته JSON به داده پایتون

```python
import json

json_text = '{"name": "Sara", "active": true}'
data = json.loads(json_text)

print(data["name"])
print(data["active"])
```

`json.loads()` یک رشته JSON را به داده پایتون تبدیل می‌کند.

### ذخیره مستقیم در فایل

```python
import json

data = {"name": "Sara", "active": True}

with open("data.json", "w", encoding="utf-8") as file:
    json.dump(data, file, ensure_ascii=False, indent=4)
```

- `json.dump()` داده را مستقیماً در فایل می‌نویسد.
- `ensure_ascii=False` باعث می‌شود حروف فارسی در فایل به شکل خوانا ذخیره شوند.
- `indent=4` ساختار فایل را مرتب و خواناتر می‌کند.

### خواندن از فایل

```python
import json

with open("data.json", "r", encoding="utf-8") as file:
    data = json.load(file)

print(data)
```

`json.load()` محتوای فایل را خوانده و به داده پایتون تبدیل می‌کند.

## ۳. مدیریت فایل ناموجود یا خراب

در اجرای اول برنامه ممکن است هنوز فایل ذخیره‌سازی وجود نداشته باشد. همچنین ممکن است فایل JSON به‌صورت دستی تغییر کرده یا خراب شده باشد.

```python
import json
from pathlib import Path

DATA_FILE = Path("tasks.json")

def load_tasks():
    if not DATA_FILE.exists():
        return []

    try:
        with DATA_FILE.open("r", encoding="utf-8") as file:
            data = json.load(file)

        if not isinstance(data, list):
            raise ValueError("The JSON root must be a list.")

        return data

    except (OSError, json.JSONDecodeError, ValueError):
        return []
```

در این نمونه، اگر فایل وجود نداشته باشد یا ساختار آن معتبر نباشد، یک فهرست خالی برمی‌گردانیم. در برنامه‌های مهم‌تر بهتر است خطا به کاربر اطلاع داده شود و فایل خراب بدون اطلاع او جایگزین نشود.

## ۴. طراحی برنامه To-Do List

امکانات برنامه:

1. افزودن کار جدید
2. نمایش فهرست کارها
3. علامت‌گذاری کار به‌عنوان انجام‌شده یا انجام‌نشده
4. حذف کار انتخاب‌شده
5. ذخیره خودکار اطلاعات در فایل JSON
6. بارگذاری اطلاعات هنگام اجرای برنامه

برای ساده ماندن پروژه، اطلاعات در فایل `tasks.json` ذخیره می‌شوند. این فایل در پوشه کاری فعلی برنامه ایجاد خواهد شد.

## ۵. کد کامل پروژه

کد زیر را در فایلی به نام `todo_app.py` ذخیره کنید. توضیحات و رشته‌های داخل کد انگلیسی هستند.

```python
import json
import tkinter as tk
from pathlib import Path
from tkinter import messagebox


DATA_FILE = Path("tasks.json")


class TodoApp:
    def __init__(self, root):
        self.root = root
        self.root.title("To-Do List")
        self.root.geometry("520x460")
        self.root.minsize(400, 350)

        self.tasks = self.load_tasks()

        self._build_ui()
        self.refresh_list()

        self.root.protocol("WM_DELETE_WINDOW", self.close_app)

    def _build_ui(self):
        input_frame = tk.Frame(self.root)
        input_frame.pack(fill="x", padx=12, pady=12)

        self.task_entry = tk.Entry(input_frame)
        self.task_entry.pack(side="left", fill="x", expand=True, padx=(0, 8))
        self.task_entry.bind("<Return>", self.add_task)

        add_button = tk.Button(
            input_frame,
            text="Add Task",
            command=self.add_task
        )
        add_button.pack(side="left")

        self.task_list = tk.Listbox(
            self.root,
            font=("Arial", 11),
            activestyle="dotbox"
        )
        self.task_list.pack(fill="both", expand=True, padx=12, pady=5)

        button_frame = tk.Frame(self.root)
        button_frame.pack(fill="x", padx=12, pady=12)

        tk.Button(
            button_frame,
            text="Toggle Done",
            command=self.toggle_task
        ).pack(side="left", padx=(0, 6))

        tk.Button(
            button_frame,
            text="Delete Task",
            command=self.delete_task
        ).pack(side="left", padx=6)

        tk.Button(
            button_frame,
            text="Save Now",
            command=self.save_tasks
        ).pack(side="right")

        self.status_label = tk.Label(
            self.root,
            text="Ready",
            anchor="w"
        )
        self.status_label.pack(fill="x", padx=12, pady=(0, 8))

    def load_tasks(self):
        if not DATA_FILE.exists():
            return []

        try:
            with DATA_FILE.open("r", encoding="utf-8") as file:
                data = json.load(file)

            if not isinstance(data, list):
                raise ValueError("The JSON root must be a list.")

            valid_tasks = []
            for item in data:
                if (
                    isinstance(item, dict)
                    and isinstance(item.get("title"), str)
                    and isinstance(item.get("done"), bool)
                ):
                    valid_tasks.append({
                        "title": item["title"],
                        "done": item["done"]
                    })

            return valid_tasks

        except (OSError, json.JSONDecodeError, ValueError) as error:
            messagebox.showwarning(
                "Load Warning",
                f"Could not load tasks.json.\n{error}"
            )
            return []

    def save_tasks(self):
        try:
            with DATA_FILE.open("w", encoding="utf-8") as file:
                json.dump(
                    self.tasks,
                    file,
                    ensure_ascii=False,
                    indent=4
                )
        except OSError as error:
            messagebox.showerror(
                "Save Error",
                f"Could not save tasks.\n{error}"
            )
            self.status_label.config(text="Save failed")
            return False

        self.status_label.config(text="Tasks saved")
        return True

    def refresh_list(self):
        self.task_list.delete(0, tk.END)

        for task in self.tasks:
            marker = "[x]" if task["done"] else "[ ]"
            self.task_list.insert(tk.END, f"{marker} {task['title']}")

    def add_task(self, event=None):
        title = self.task_entry.get().strip()

        if not title:
            messagebox.showwarning(
                "Empty Task",
                "Please enter a task title."
            )
            return

        self.tasks.append({
            "title": title,
            "done": False
        })

        self.task_entry.delete(0, tk.END)
        self.refresh_list()

        if self.save_tasks():
            self.status_label.config(text="Task added")

    def get_selected_index(self):
        selection = self.task_list.curselection()

        if not selection:
            messagebox.showinfo(
                "No Selection",
                "Please select a task first."
            )
            return None

        return selection[0]

    def toggle_task(self):
        index = self.get_selected_index()
        if index is None:
            return

        self.tasks[index]["done"] = not self.tasks[index]["done"]
        self.refresh_list()
        self.task_list.selection_set(index)

        if self.save_tasks():
            self.status_label.config(text="Task updated")

    def delete_task(self):
        index = self.get_selected_index()
        if index is None:
            return

        confirmed = messagebox.askyesno(
            "Delete Task",
            "Do you want to delete the selected task?"
        )

        if not confirmed:
            return

        del self.tasks[index]
        self.refresh_list()

        if self.save_tasks():
            self.status_label.config(text="Task deleted")

    def close_app(self):
        if self.save_tasks():
            self.root.destroy()


if __name__ == "__main__":
    root = tk.Tk()
    app = TodoApp(root)
    root.mainloop()
```

## ۶. توضیح معماری برنامه

### چرا اطلاعات را در `self.tasks` نگه می‌داریم؟

`self.tasks` فهرست اصلی اطلاعات برنامه است. هر کار یک دیکشنری با دو کلید دارد:

```python
{
    "title": "Practice Python",
    "done": False
}
```

`Listbox` فقط نمایش کارها را بر عهده دارد. اطلاعات اصلی در `self.tasks` نگهداری می‌شود و پس از هر تغییر، تابع `refresh_list()` نمایش را با آن هماهنگ می‌کند.

### چرا `refresh_list()` را جدا نوشته‌ایم؟

اگر کد نمایش فهرست در چند تابع تکرار شود، تغییر ظاهر برنامه سخت‌تر می‌شود. با یک تابع مشترک، هر بار که کار اضافه، حذف یا ویرایش می‌شود، کافی است آن را صدا بزنیم.

### چرا `save_tasks()` مقدار `True` یا `False` برمی‌گرداند؟

ذخیره فایل ممکن است شکست بخورد. مقدار بازگشتی به بخش‌های دیگر برنامه اجازه می‌دهد بفهمند عملیات ذخیره موفق بوده یا نه.

### تفاوت `dump()` و `load()` با `dumps()` و `loads()`

| تابع | کاربرد |
|---|---|
| `json.dump(data, file)` | نوشتن داده در فایل |
| `json.load(file)` | خواندن داده از فایل |
| `json.dumps(data)` | تبدیل داده به رشته JSON |
| `json.loads(text)` | تبدیل رشته JSON به داده پایتون |

## ۷. تمرین‌های دانشجو

### تمرین ۱ — پاک کردن ورودی بعد از افزودن کار

پس از افزودن کار، کادر ورودی باید خالی شود.

**پاسخ:**

```python
self.task_entry.delete(0, tk.END)
```

این خط در پروژه پس از افزودن موفق کار قرار دارد.

### تمرین ۲ — نمایش تعداد کارها

تابعی بنویسید که تعداد کل کارها را در یک پیام نمایش دهد.

**پاسخ:**

```python
def show_task_count(self):
    total = len(self.tasks)
    completed = sum(1 for task in self.tasks if task["done"])

    messagebox.showinfo(
        "Task Count",
        f"Total: {total}\nCompleted: {completed}"
    )
```

برای استفاده، یک دکمه یا گزینه منو با `command=self.show_task_count` اضافه کنید.

### تمرین ۳ — جلوگیری از افزودن کار تکراری

پیش از افزودن کار جدید، بررسی کنید آیا عنوان مشابهی در فهرست وجود دارد یا نه.

**پاسخ پیشنهادی:**

```python
normalized_title = title.casefold()

for task in self.tasks:
    if task["title"].casefold() == normalized_title:
        messagebox.showwarning(
            "Duplicate Task",
            "This task already exists."
        )
        return
```

این بخش را بعد از بررسی خالی نبودن `title` و قبل از `self.tasks.append(...)` قرار دهید.

### تمرین ۴ — پاک کردن همه کارها

یک تابع بسازید که پس از تأیید کاربر، تمام کارها را حذف کند.

**پاسخ:**

```python
def clear_all_tasks(self):
    confirmed = messagebox.askyesno(
        "Clear All",
        "Do you want to delete all tasks?"
    )

    if not confirmed:
        return

    self.tasks.clear()
    self.refresh_list()
    self.save_tasks()
```

یک دکمه با `command=self.clear_all_tasks` اضافه کنید. در برنامه‌های واقعی بهتر است پیش از پاک کردن، وجود نسخه پشتیبان یا امکان بازگشت را نیز در نظر بگیریم.

## ۸. خطاهای رایج

- **فراموش کردن ذخیره پس از تغییر:** اگر پس از افزودن یا حذف، `save_tasks()` اجرا نشود، تغییرات ممکن است پس از بسته شدن برنامه از بین بروند.
- **استفاده از `json.load()` برای رشته:** این تابع فایل‌مانند می‌خواهد؛ برای رشته JSON از `json.loads()` استفاده کنید.
- **استفاده از `json.loads()` برای فایل:** برای خواندن مستقیم از فایل، `json.load()` مناسب است.
- **نادیده گرفتن فایل ناموجود:** در اولین اجرا ممکن است `tasks.json` هنوز وجود نداشته باشد.
- **انتخاب نکردن یک ردیف:** پیش از استفاده از اندیس انتخاب‌شده در `Listbox`، نتیجه `curselection()` را بررسی کنید.
- **ذخیره اطلاعات فارسی:** هنگام باز کردن فایل از `encoding="utf-8"` و هنگام ذخیره JSON از `ensure_ascii=False` استفاده کنید.
- **اشتباه گرفتن نمایش و داده اصلی:** `Listbox` فقط نمایش است؛ داده‌ها در `self.tasks` نگهداری می‌شوند.

## ۹. جمع‌بندی جلسه

در این جلسه یاد گرفتیم:

- JSON قالبی ساده برای ذخیره داده‌های ساخت‌یافته است.
- `json.dump()` و `json.load()` برای کار با فایل JSON استفاده می‌شوند.
- می‌توان وضعیت برنامه را در یک فایل نگهداری کرد تا پس از اجرای دوباره باقی بماند.
- بهتر است داده اصلی و بخش نمایش از هم جدا باشند.
- توابع کوچک و مشخص، توسعه و عیب‌یابی برنامه را آسان‌تر می‌کنند.

## ۱۰. چک‌لیست مدرس

- [ ] تفاوت JSON و دیکشنری پایتون را توضیح دادم.
- [ ] `dump`، `load`، `dumps` و `loads` را مقایسه کردم.
- [ ] برنامه را اجرا کردم و یک کار افزودم.
- [ ] برنامه را بستم و دوباره باز کردم تا ماندگاری داده بررسی شود.
- [ ] کار را انجام‌شده علامت زدم و آن را حذف کردم.
- [ ] فایل JSON تولیدشده را با یک ویرایشگر متن باز کردم.

## جلسه بعد

در جلسه ۹ می‌توانیم برنامه را بهبود دهیم: جست‌وجو و فیلتر کارها، نمایش کارهای انجام‌شده و انجام‌نشده، و مرتب‌سازی فهرست.
