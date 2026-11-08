---
title: "Tkinter - جلسه ۹ - جست‌وجو، فیلتر و مرتب‌سازی"
aliases:
  - Tkinter Session 9
  - To-Do Search Filter Sort
tags:
  - Python
  - Tkinter
  - ToDo
  - آموزش
level: intermediate
status: ready
---

# جلسه ۹ — جست‌وجو، فیلتر و مرتب‌سازی در برنامه To-Do

## هدف جلسه

در جلسه قبل برنامه To-Do ساختیم و اطلاعات را در فایل JSON ذخیره کردیم. در این جلسه همان برنامه را کاربردی‌تر می‌کنیم:

- جست‌وجو در عنوان کارها
- نمایش همه کارها، کارهای انجام‌شده یا کارهای باقی‌مانده
- مرتب‌سازی بر اساس عنوان
- حفظ انتخاب و همگام نگه داشتن نمایش با داده‌های اصلی
- ذخیره تغییرات در JSON

نکته مهم این جلسه این است که **فهرست اصلی داده‌ها را از فهرست نمایشی جدا نگه می‌داریم**. جست‌وجو و فیلتر نباید اطلاعات اصلی را حذف کنند؛ آن‌ها فقط تعیین می‌کنند کدام ردیف‌ها نمایش داده شوند.

## پیش‌نیازها

- کلاس و تابع در پایتون
- `Listbox` و `Entry`
- `StringVar` و `Combobox` در صورت استفاده از نسخه توسعه‌یافته
- ذخیره و خواندن JSON
- برنامه جلسه ۸

## ۱. داده اصلی و داده نمایشی

فرض کنید فهرست اصلی چنین باشد:

```python
tasks = [
    {"title": "Read Python notes", "done": False},
    {"title": "Practice Tkinter", "done": True},
    {"title": "Build a project", "done": False}
]
```

اگر کاربر فقط کارهای انجام‌شده را بخواهد ببیند، نباید دو کار دیگر را از `tasks` حذف کنیم. باید یک فهرست نمایشی بسازیم که فقط کارهای منطبق با فیلتر را در خود داشته باشد.

در این پروژه دو فهرست خواهیم داشت:

- `self.tasks`: داده‌های اصلی
- `self.visible_indices`: اندیس‌های داده‌های اصلی که اکنون در `Listbox` دیده می‌شوند

این روش کمک می‌کند اگر کاربر یک ردیف فیلترشده را انتخاب کرد، بتوانیم کار درست را در فهرست اصلی پیدا کنیم.

## ۲. افزودن کادر جست‌وجو و فیلتر

در رابط کاربری، بالای `Listbox` یک `Entry` برای جست‌وجو و یک `Combobox` برای انتخاب وضعیت قرار می‌دهیم.

گزینه‌های فیلتر:

- All tasks
- Pending
- Completed

برای تشخیص تغییر متن جست‌وجو از رویداد `KeyRelease` استفاده می‌کنیم. این رویداد پس از رها شدن کلید اجرا می‌شود.

```python
self.search_entry.bind("<KeyRelease>", self.refresh_list)
```

در برنامه کامل، تابع `refresh_list()` متن جست‌وجو و وضعیت انتخاب‌شده را بررسی می‌کند.

## ۳. مرتب‌سازی

برای مرتب‌سازی فهرست اصلی می‌توانیم از `sorted()` استفاده کنیم:

```python
sorted_tasks = sorted(
    self.tasks,
    key=lambda task: task["title"].casefold()
)
```

`casefold()` برای مقایسه بدون حساسیت به بزرگی و کوچکی حروف انگلیسی مفید است. مرتب‌سازی با آن جایگزین کامل قواعد مرتب‌سازی زبان فارسی نیست، اما برای این پروژه ساده کافی است.

در این پروژه مرتب‌سازی را فقط روی ترتیب نمایش اعمال می‌کنیم؛ ترتیب اصلی `self.tasks` تغییر نمی‌کند.

## ۴. کد کامل پروژه

این نسخه مستقل است و می‌تواند جایگزین کد جلسه ۸ شود. آن را با نام `todo_search_app.py` ذخیره کنید. اگر فایل `tasks.json` مربوط به جلسه قبل در پوشه کاری برنامه باشد، برنامه تلاش می‌کند اطلاعات آن را بارگذاری کند.

```python
import json
import tkinter as tk
from pathlib import Path
from tkinter import messagebox, ttk


DATA_FILE = Path("tasks.json")


class TodoApp:
    def __init__(self, root):
        self.root = root
        self.root.title("To-Do List")
        self.root.geometry("620x520")
        self.root.minsize(460, 380)

        self.tasks = self.load_tasks()
        self.visible_indices = []

        self.search_var = tk.StringVar()
        self.filter_var = tk.StringVar(value="All tasks")
        self.sort_var = tk.StringVar(value="Original order")

        self._build_ui()
        self.refresh_list()
        self.root.protocol("WM_DELETE_WINDOW", self.close_app)

    def _build_ui(self):
        input_frame = tk.Frame(self.root)
        input_frame.pack(fill="x", padx=12, pady=(12, 6))

        self.task_entry = tk.Entry(input_frame)
        self.task_entry.pack(side="left", fill="x", expand=True, padx=(0, 8))
        self.task_entry.bind("<Return>", self.add_task)

        tk.Button(
            input_frame,
            text="Add Task",
            command=self.add_task
        ).pack(side="left")

        search_frame = tk.Frame(self.root)
        search_frame.pack(fill="x", padx=12, pady=6)

        tk.Label(search_frame, text="Search:").pack(side="left", padx=(0, 6))

        self.search_entry = tk.Entry(
            search_frame,
            textvariable=self.search_var
        )
        self.search_entry.pack(side="left", fill="x", expand=True)
        self.search_entry.bind("<KeyRelease>", self.refresh_list)

        options_frame = tk.Frame(self.root)
        options_frame.pack(fill="x", padx=12, pady=6)

        tk.Label(options_frame, text="Status:").pack(side="left")

        self.filter_box = ttk.Combobox(
            options_frame,
            textvariable=self.filter_var,
            values=["All tasks", "Pending", "Completed"],
            state="readonly",
            width=14
        )
        self.filter_box.pack(side="left", padx=(6, 16))
        self.filter_box.bind("<<ComboboxSelected>>", self.refresh_list)

        tk.Label(options_frame, text="Sort:").pack(side="left")

        self.sort_box = ttk.Combobox(
            options_frame,
            textvariable=self.sort_var,
            values=["Original order", "Title A-Z", "Title Z-A"],
            state="readonly",
            width=16
        )
        self.sort_box.pack(side="left", padx=6)
        self.sort_box.bind("<<ComboboxSelected>>", self.refresh_list)

        self.task_list = tk.Listbox(
            self.root,
            font=("Arial", 11),
            activestyle="dotbox"
        )
        self.task_list.pack(fill="both", expand=True, padx=12, pady=6)

        button_frame = tk.Frame(self.root)
        button_frame.pack(fill="x", padx=12, pady=8)

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
            text="Clear Search",
            command=self.clear_search
        ).pack(side="left", padx=6)

        self.status_label = tk.Label(self.root, text="Ready", anchor="w")
        self.status_label.pack(fill="x", padx=12, pady=(0, 8))

    def load_tasks(self):
        if not DATA_FILE.exists():
            return []

        try:
            with DATA_FILE.open("r", encoding="utf-8") as file:
                data = json.load(file)

            if not isinstance(data, list):
                raise ValueError("The JSON root must be a list.")

            tasks = []
            for item in data:
                if (
                    isinstance(item, dict)
                    and isinstance(item.get("title"), str)
                    and isinstance(item.get("done"), bool)
                ):
                    tasks.append({
                        "title": item["title"],
                        "done": item["done"]
                    })

            return tasks

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
            messagebox.showerror("Save Error", str(error))
            self.status_label.config(text="Save failed")
            return False

        self.status_label.config(text="Tasks saved")
        return True

    def refresh_list(self, event=None):
        query = self.search_var.get().strip().casefold()
        status_filter = self.filter_var.get()
        sort_option = self.sort_var.get()

        matched_indices = []

        for index, task in enumerate(self.tasks):
            title_matches = query in task["title"].casefold()

            if status_filter == "Pending":
                status_matches = not task["done"]
            elif status_filter == "Completed":
                status_matches = task["done"]
            else:
                status_matches = True

            if title_matches and status_matches:
                matched_indices.append(index)

        if sort_option == "Title A-Z":
            matched_indices.sort(
                key=lambda index: self.tasks[index]["title"].casefold()
            )
        elif sort_option == "Title Z-A":
            matched_indices.sort(
                key=lambda index: self.tasks[index]["title"].casefold(),
                reverse=True
            )

        self.visible_indices = matched_indices
        self.task_list.delete(0, tk.END)

        for index in self.visible_indices:
            task = self.tasks[index]
            marker = "[x]" if task["done"] else "[ ]"
            self.task_list.insert(tk.END, f"{marker} {task['title']}")

        self.status_label.config(
            text=f"Showing {len(self.visible_indices)} of {len(self.tasks)} tasks"
        )

    def add_task(self, event=None):
        title = self.task_entry.get().strip()

        if not title:
            messagebox.showwarning("Empty Task", "Please enter a task title.")
            return

        if any(task["title"].casefold() == title.casefold()
               for task in self.tasks):
            messagebox.showwarning(
                "Duplicate Task",
                "This task already exists."
            )
            return

        self.tasks.append({"title": title, "done": False})
        self.task_entry.delete(0, tk.END)

        self.refresh_list()
        if self.save_tasks():
            self.status_label.config(text="Task added")

    def get_selected_task_index(self):
        selection = self.task_list.curselection()

        if not selection:
            messagebox.showinfo(
                "No Selection",
                "Please select a task first."
            )
            return None

        visible_row = selection[0]
        return self.visible_indices[visible_row]

    def toggle_task(self):
        task_index = self.get_selected_task_index()
        if task_index is None:
            return

        self.tasks[task_index]["done"] = not self.tasks[task_index]["done"]
        self.refresh_list()
        self.save_tasks()

    def delete_task(self):
        task_index = self.get_selected_task_index()
        if task_index is None:
            return

        confirmed = messagebox.askyesno(
            "Delete Task",
            "Delete the selected task?"
        )
        if not confirmed:
            return

        del self.tasks[task_index]
        self.refresh_list()
        self.save_tasks()

    def clear_search(self):
        self.search_var.set("")
        self.refresh_list()
        self.search_entry.focus_set()

    def close_app(self):
        if self.save_tasks():
            self.root.destroy()


if __name__ == "__main__":
    root = tk.Tk()
    app = TodoApp(root)
    root.mainloop()
```

## ۵. بررسی منطق جست‌وجو و فیلتر

### جست‌وجو چگونه کار می‌کند؟

```python
query = self.search_var.get().strip().casefold()
title_matches = query in task["title"].casefold()
```

- `get()` مقدار کادر جست‌وجو را می‌خواند.
- `strip()` فاصله‌های ابتدا و انتها را حذف می‌کند.
- `casefold()` مقایسه متن انگلیسی را نسبت به بزرگی و کوچکی حروف بی‌تفاوت می‌کند.
- عملگر `in` بررسی می‌کند که عبارت جست‌وجو داخل عنوان کار وجود دارد یا خیر.

اگر عبارت جست‌وجو خالی باشد، رشته خالی داخل هر رشته‌ای پیدا می‌شود؛ در نتیجه همه عنوان‌ها از نظر جست‌وجو مطابقت خواهند داشت.

### چرا `visible_indices` لازم است؟

فرض کنید فهرست اصلی سه کار دارد، اما جست‌وجو فقط کار سوم را نمایش می‌دهد. در `Listbox`، این کار در ردیف صفر دیده می‌شود، ولی اندیس آن در `self.tasks` برابر ۲ است.

پس نباید مستقیماً از شماره ردیف انتخاب‌شده در `Listbox` برای تغییر `self.tasks` استفاده کنیم. ابتدا باید ردیف نمایشی را به اندیس اصلی تبدیل کنیم:

```python
visible_row = selection[0]
task_index = self.visible_indices[visible_row]
```

این یکی از مهم‌ترین نکته‌های طراحی برنامه‌های دارای جست‌وجو و فیلتر است.

## ۶. تمرین‌های دانشجو

### تمرین ۱ — فیلتر کارهای انجام‌نشده

گزینه‌ای به فیلتر اضافه کنید که فقط کارهای انجام‌نشده را نمایش دهد.

**راهنمای پاسخ:** گزینه `Pending` و شرط `not task["done"]` در پروژه وجود دارد. دانشجو باید شرط را پیدا کرده و توضیح دهد.

### تمرین ۲ — شمارش کارهای انجام‌شده

تابعی بنویسید که تعداد کل کارها و تعداد کارهای انجام‌شده را نمایش دهد.

**پاسخ:**

```python
def show_counts(self):
    total = len(self.tasks)
    completed = sum(1 for task in self.tasks if task["done"])
    pending = total - completed

    messagebox.showinfo(
        "Task Summary",
        f"Total: {total}\nCompleted: {completed}\nPending: {pending}"
    )
```

برای استفاده از تابع، یک دکمه با `command=self.show_counts` اضافه کنید.

### تمرین ۳ — جست‌وجو با دکمه

به‌جای جست‌وجوی زنده، یک دکمه Search اضافه کنید که با کلیک روی آن، فهرست به‌روزرسانی شود.

**پاسخ:** رویداد زیر را از ساخت رابط حذف کنید:

```python
self.search_entry.bind("<KeyRelease>", self.refresh_list)
```

سپس دکمه‌ای بسازید:

```python
tk.Button(
    search_frame,
    text="Search",
    command=self.refresh_list
).pack(side="left", padx=6)
```

### تمرین ۴ — پاک کردن فیلتر

دکمه‌ای اضافه کنید که جست‌وجو را پاک کند و فیلتر را به `All tasks` برگرداند.

**پاسخ:**

```python
def reset_filters(self):
    self.search_var.set("")
    self.filter_var.set("All tasks")
    self.sort_var.set("Original order")
    self.refresh_list()
```

یک دکمه با `command=self.reset_filters` اضافه کنید.

### تمرین ۵ — شمارش نتایج

پس از جست‌وجو، تعداد نتیجه‌های نمایش‌داده‌شده و تعداد کل کارها را در `status_label` نشان دهید.

**پاسخ:** در انتهای `refresh_list()` از این دستور استفاده شده است:

```python
self.status_label.config(
    text=f"Showing {len(self.visible_indices)} of {len(self.tasks)} tasks"
)
```

دانشجو باید توضیح دهد چرا تعداد `visible_indices` ممکن است کمتر از تعداد `tasks` باشد.

## ۷. خطاهای رایج

- **حذف اشتباهی داده‌ها هنگام فیلتر:** فیلتر باید فقط نمایش را تغییر دهد، نه فهرست اصلی را.
- **استفاده از اندیس نمایشی برای تغییر داده اصلی:** باید از `visible_indices` برای پیدا کردن کار واقعی استفاده شود.
- **فراموش کردن به‌روزرسانی `Listbox`:** پس از افزودن، حذف یا تغییر وضعیت، باید `refresh_list()` اجرا شود.
- **مرتب‌سازی فهرست اصلی ناخواسته:** در این پروژه ترتیب نمایش تغییر می‌کند و ترتیب اصلی حفظ می‌شود.
- **فراموش کردن ذخیره JSON:** پس از تغییر داده اصلی، `save_tasks()` باید اجرا شود.
- **انتظار مرتب‌سازی فارسی کامل:** `casefold()` برای مقایسه بدون حساسیت به بزرگی و کوچکی حروف مفید است، اما قواعد تخصصی مرتب‌سازی زبان فارسی را پیاده‌سازی نمی‌کند.

## ۸. جمع‌بندی جلسه

- با جست‌وجو می‌توانیم فقط داده‌های موردنظر را نمایش دهیم.
- فیلتر بر اساس وضعیت، کارهای انجام‌شده و انجام‌نشده را جدا می‌کند.
- مرتب‌سازی می‌تواند فقط ترتیب نمایش را تغییر دهد.
- جدا نگه داشتن داده اصلی و داده نمایشی از خطا در ویرایش یا حذف جلوگیری می‌کند.
- پس از تغییر داده‌ها، باید نمایش و فایل JSON را به‌روزرسانی کنیم.

## ۹. چک‌لیست مدرس

- [ ] جست‌وجوی زنده را اجرا کردم.
- [ ] فیلتر All tasks، Pending و Completed را آزمایش کردم.
- [ ] مرتب‌سازی A-Z و Z-A را نشان دادم.
- [ ] حذف یک کار در حالت فیلترشده را آزمایش کردم.
- [ ] تفاوت اندیس ردیف نمایشی و اندیس داده اصلی را توضیح دادم.
- [ ] ذخیره و بارگذاری مجدد فایل JSON را بررسی کردم.

## جلسه بعد

در جلسه ۱۰ می‌توانیم رابط کاربری را منظم‌تر کنیم: استفاده هدفمند از `ttk`، چیدمان بهتر، نوار ابزار و جداسازی کد رابط کاربری از منطق برنامه.
