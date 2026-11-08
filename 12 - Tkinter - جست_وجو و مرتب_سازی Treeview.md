---
title: "Tkinter - جلسه ۱۲ - جست‌وجو و مرتب‌سازی Treeview"
aliases:
  - Tkinter Session 12
  - Treeview Search and Sorting
tags:
  - Python
  - Tkinter
  - ttk
  - Treeview
  - آموزش
level: intermediate
status: ready
---

# جلسه ۱۲ — جست‌وجو، فیلتر و مرتب‌سازی جدول

## هدف جلسه

در جلسه قبل با `ttk.Treeview` آشنا شدیم و برنامه مدیریت دانشجو ساختیم. در این جلسه همان جدول را کاربردی‌تر می‌کنیم تا کاربر بتواند اطلاعات را جست‌وجو کند، بر اساس یک معیار فیلتر کند و با کلیک روی عنوان ستون‌ها داده‌ها را مرتب کند.

در پایان جلسه می‌توانیم:

- داده‌ها را با متن جست‌وجو کنیم.
- نتایج را بر اساس یک فیلد فیلتر کنیم.
- ستون‌های جدول را به ترتیب صعودی یا نزولی مرتب کنیم.
- داده‌های اصلی را از داده‌های نمایشی جدا نگه داریم.
- پس از تغییر داده‌ها، جدول را دوباره از روی داده‌های اصلی بسازیم.

## ۱. اصل مهم: داده‌های اصلی و جدول نمایشی

در برنامه‌های جدی بهتر است اطلاعات اصلی را در یک فهرست پایتون نگهداری کنیم و `Treeview` را فقط برای نمایش آن اطلاعات به کار ببریم.

ساختار نمونه:

```python
students = [
    {"id": 1, "name": "Sara", "age": 20, "city": "Tehran"},
    {"id": 2, "name": "Amir", "age": 22, "city": "Tabriz"},
    {"id": 3, "name": "Neda", "age": 19, "city": "Shiraz"}
]
```

مزیت این روش چیست؟ اگر فقط ردیف‌های جدول را جست‌وجو یا مرتب کنیم، ممکن است مدیریت اطلاعات در ادامه دشوار شود. داشتن یک منبع داده مشخص، تغییرات را قابل‌پیش‌بینی‌تر می‌کند.

در این جلسه، `students` منبع داده اصلی است و جدول از روی آن بازسازی می‌شود.

## ۲. ساخت جدول و نمایش داده‌ها

```python
import tkinter as tk
from tkinter import ttk

root = tk.Tk()
root.title("Student Table")
root.geometry("650x350")

students = [
    {"id": 1, "name": "Sara", "age": 20, "city": "Tehran"},
    {"id": 2, "name": "Amir", "age": 22, "city": "Tabriz"},
    {"id": 3, "name": "Neda", "age": 19, "city": "Shiraz"}
]

columns = ("id", "name", "age", "city")
tree = ttk.Treeview(root, columns=columns, show="headings")

for column in columns:
    tree.heading(column, text=column.title())

tree.pack(fill="both", expand=True, padx=10, pady=10)

def refresh_table(rows):
    tree.delete(*tree.get_children())

    for student in rows:
        tree.insert(
            "",
            tk.END,
            iid=str(student["id"]),
            values=(
                student["id"],
                student["name"],
                student["age"],
                student["city"]
            )
        )

refresh_table(students)
root.mainloop()
```

### توضیح `refresh_table()`

- `tree.get_children()` شناسه ردیف‌های موجود را برمی‌گرداند.
- `tree.delete(*tree.get_children())` همه ردیف‌های فعلی را حذف می‌کند.
- سپس ردیف‌های موردنظر دوباره وارد جدول می‌شوند.

علامت `*` در اینجا عناصر یک مجموعه را به آرگومان‌های جداگانه تبدیل می‌کند. اگر جدول خالی باشد، این دستور نیز بدون مشکل اجرا می‌شود.

## ۳. جست‌وجو در اطلاعات

فرض کنید کاربر بخواهد نام یا شهر دانشجو را جست‌وجو کند. از یک `Entry` برای دریافت عبارت و از تابع `filter_students()` برای انتخاب نتایج استفاده می‌کنیم.

```python
def filter_students():
    query = search_entry.get().strip().casefold()

    filtered = [
        student for student in students
        if query in student["name"].casefold()
        or query in student["city"].casefold()
    ]

    refresh_table(filtered)
```

`casefold()` مقایسه متن را برای جست‌وجوی بدون حساسیت به بزرگی و کوچکی حروف مناسب‌تر می‌کند. برای متن فارسی نیز کار می‌کند، هرچند در فارسی تفاوت حروفی مانند «ی» و «ي» یا «ک» و «ك» را خودبه‌خود یکسان نمی‌کند.

برای اجرای جست‌وجو هنگام تایپ:

```python
search_entry.bind("<KeyRelease>", lambda event: filter_students())
```

رویداد `KeyRelease` پس از رها کردن کلید اجرا می‌شود؛ در این زمان متن ورودی به‌روزرسانی شده است.

## ۴. فیلتر بر اساس شهر

فیلتر با جست‌وجو فرق دارد. جست‌وجو می‌تواند بخشی از متن را پیدا کند؛ فیلتر معمولاً بر اساس یک شرط مشخص، مانند شهر، داده‌ها را محدود می‌کند.

```python
def filter_by_city(city):
    if city == "All":
        refresh_table(students)
        return

    filtered = [
        student for student in students
        if student["city"] == city
    ]
    refresh_table(filtered)
```

برای انتخاب شهر می‌توان از `ttk.Combobox` استفاده کرد:

```python
city_combo = ttk.Combobox(
    root,
    values=("All", "Tehran", "Tabriz", "Shiraz"),
    state="readonly"
)
city_combo.set("All")
city_combo.bind(
    "<<ComboboxSelected>>",
    lambda event: filter_by_city(city_combo.get())
)
```

این نمونه فیلتر شهر را جداگانه نشان می‌دهد. در پروژه کامل، جست‌وجو و فیلتر شهر با هم ترکیب می‌شوند.

## ۵. مرتب‌سازی با کلیک روی عنوان ستون

می‌توانیم با کلیک روی سرستون، اطلاعات را بر اساس همان ستون مرتب کنیم. برای ستون سن باید مقدار عددی را با عدد مقایسه کنیم، نه به‌عنوان متن.

```python
def sort_students(column, reverse=False):
    if column == "age":
        sorted_students = sorted(
            students,
            key=lambda student: student["age"],
            reverse=reverse
        )
    else:
        sorted_students = sorted(
            students,
            key=lambda student: str(student[column]).casefold(),
            reverse=reverse
        )

    refresh_table(sorted_students)
    tree.heading(
        column,
        command=lambda: sort_students(column, not reverse)
    )
```

هنگام ساخت عنوان ستون‌ها، برای هر عنوان فرمان مرتب‌سازی را تنظیم کنید:

```python
for column in columns:
    tree.heading(
        column,
        text=column.title(),
        command=lambda col=column: sort_students(col)
    )
```

نکته مهم: `lambda col=column` مقدار فعلی ستون را ذخیره می‌کند. اگر فقط `lambda: sort_students(column)` بنویسیم، ممکن است همه عنوان‌ها در زمان کلیک به آخرین مقدار متغیر حلقه اشاره کنند.

این نمونه ساده است؛ در پروژه کامل، مرتب‌سازی بر اساس داده‌های فیلترشده انجام می‌شود و جهت مرتب‌سازی هر ستون نگهداری می‌شود.

## ۶. پروژه کامل: جدول دانشجو با جست‌وجو، فیلتر و مرتب‌سازی

کد زیر مستقل است و می‌توانید آن را در فایلی مانند `student_table.py` ذخیره و اجرا کنید. داده‌ها تا زمانی که برنامه باز است در حافظه می‌مانند؛ هنوز ذخیره‌سازی دائمی نداریم.

```python
import tkinter as tk
from tkinter import ttk, messagebox


class StudentTableApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Student Table")
        self.root.geometry("820x520")
        self.root.minsize(680, 420)

        self.students = [
            {"id": 1, "name": "Sara", "age": 20, "city": "Tehran"},
            {"id": 2, "name": "Amir", "age": 22, "city": "Tabriz"},
            {"id": 3, "name": "Neda", "age": 19, "city": "Shiraz"},
            {"id": 4, "name": "Reza", "age": 21, "city": "Tehran"},
            {"id": 5, "name": "Mina", "age": 23, "city": "Isfahan"},
        ]

        self.next_id = 6
        self.sort_state = {}
        self._build_ui()
        self.refresh_table()

    def _build_ui(self):
        search_frame = ttk.LabelFrame(
            self.root,
            text="Search and Filter",
            padding=10
        )
        search_frame.pack(fill="x", padx=12, pady=12)

        ttk.Label(search_frame, text="Search name or city:").grid(
            row=0, column=0, sticky="w", padx=5, pady=5
        )

        self.search_entry = ttk.Entry(search_frame)
        self.search_entry.grid(
            row=0, column=1, sticky="ew", padx=5, pady=5
        )
        self.search_entry.bind(
            "<KeyRelease>",
            lambda event: self.refresh_table()
        )

        ttk.Label(search_frame, text="City:").grid(
            row=0, column=2, sticky="w", padx=5, pady=5
        )

        self.city_combo = ttk.Combobox(
            search_frame,
            values=("All", "Tehran", "Tabriz", "Shiraz", "Isfahan"),
            state="readonly",
            width=12
        )
        self.city_combo.set("All")
        self.city_combo.grid(
            row=0, column=3, sticky="ew", padx=5, pady=5
        )
        self.city_combo.bind(
            "<<ComboboxSelected>>",
            lambda event: self.refresh_table()
        )

        ttk.Button(
            search_frame,
            text="Clear Search",
            command=self.clear_search
        ).grid(row=0, column=4, padx=5, pady=5)

        search_frame.columnconfigure(1, weight=1)

        table_frame = ttk.Frame(self.root)
        table_frame.pack(fill="both", expand=True, padx=12, pady=(0, 8))

        columns = ("id", "name", "age", "city")
        self.tree = ttk.Treeview(
            table_frame,
            columns=columns,
            show="headings",
            selectmode="browse"
        )

        headings = {
            "id": "ID",
            "name": "Name",
            "age": "Age",
            "city": "City"
        }

        for column, title in headings.items():
            self.tree.heading(
                column,
                text=title,
                command=lambda col=column: self.sort_by(col)
            )

        self.tree.column("id", width=70, anchor="center")
        self.tree.column("name", width=220)
        self.tree.column("age", width=100, anchor="center")
        self.tree.column("city", width=180)

        scrollbar = ttk.Scrollbar(
            table_frame,
            orient="vertical",
            command=self.tree.yview
        )
        self.tree.configure(yscrollcommand=scrollbar.set)

        self.tree.pack(side="left", fill="both", expand=True)
        scrollbar.pack(side="right", fill="y")

        self.status_label = ttk.Label(self.root, text="")
        self.status_label.pack(anchor="w", padx=14, pady=(0, 10))

        button_frame = ttk.Frame(self.root)
        button_frame.pack(fill="x", padx=12, pady=(0, 12))

        ttk.Button(
            button_frame,
            text="Show Selected",
            command=self.show_selected
        ).pack(side="left", padx=(0, 6))

        ttk.Button(
            button_frame,
            text="Delete Selected",
            command=self.delete_selected
        ).pack(side="left", padx=6)

        ttk.Button(
            button_frame,
            text="Add Sample",
            command=self.add_sample
        ).pack(side="left", padx=6)

    def get_filtered_students(self):
        query = self.search_entry.get().strip().casefold()
        selected_city = self.city_combo.get()

        result = []

        for student in self.students:
            matches_query = (
                query in student["name"].casefold()
                or query in student["city"].casefold()
            )
            matches_city = (
                selected_city == "All"
                or student["city"] == selected_city
            )

            if matches_query and matches_city:
                result.append(student)

        return result

    def refresh_table(self):
        rows = self.get_filtered_students()

        tree_children = self.tree.get_children()
        if tree_children:
            self.tree.delete(*tree_children)

        for student in rows:
            self.tree.insert(
                "",
                tk.END,
                iid=str(student["id"]),
                values=(
                    student["id"],
                    student["name"],
                    student["age"],
                    student["city"]
                )
            )

        self.status_label.config(
            text=f"Showing {len(rows)} of {len(self.students)} students"
        )

    def sort_by(self, column):
        reverse = self.sort_state.get(column, False)
        rows = self.get_filtered_students()

        if column == "age" or column == "id":
            rows.sort(
                key=lambda student: student[column],
                reverse=reverse
            )
        else:
            rows.sort(
                key=lambda student: student[column].casefold(),
                reverse=reverse
            )

        current_selection = self.tree.selection()

        children = self.tree.get_children()
        if children:
            self.tree.delete(*children)

        for student in rows:
            self.tree.insert(
                "",
                tk.END,
                iid=str(student["id"]),
                values=(
                    student["id"],
                    student["name"],
                    student["age"],
                    student["city"]
                )
            )

        self.sort_state[column] = not reverse
        self.status_label.config(
            text=f"Sorted by {column.title()} - {len(rows)} result(s)"
        )

        if current_selection and self.tree.exists(current_selection[0]):
            self.tree.selection_set(current_selection[0])

    def clear_search(self):
        self.search_entry.delete(0, tk.END)
        self.city_combo.set("All")
        self.refresh_table()

    def show_selected(self):
        selection = self.tree.selection()

        if not selection:
            messagebox.showinfo(
                "No Selection",
                "Select a student first."
            )
            return

        values = self.tree.item(selection[0], "values")
        messagebox.showinfo(
            "Student Details",
            f"ID: {values[0]}\nName: {values[1]}\n"
            f"Age: {values[2]}\nCity: {values[3]}"
        )

    def delete_selected(self):
        selection = self.tree.selection()

        if not selection:
            messagebox.showinfo(
                "No Selection",
                "Select a student to delete."
            )
            return

        item_id = int(selection[0])
        confirmed = messagebox.askyesno(
            "Confirm Delete",
            "Delete the selected student?"
        )

        if not confirmed:
            return

        self.students = [
            student for student in self.students
            if student["id"] != item_id
        ]

        self.refresh_table()

    def add_sample(self):
        sample_students = [
            ("Omid", 24, "Tehran"),
            ("Laleh", 20, "Shiraz"),
            ("Pouya", 22, "Isfahan")
        ]

        name, age, city = sample_students[
            (self.next_id - 6) % len(sample_students)
        ]

        self.students.append({
            "id": self.next_id,
            "name": name,
            "age": age,
            "city": city
        })
        self.next_id += 1
        self.refresh_table()


if __name__ == "__main__":
    root = tk.Tk()
    app = StudentTableApp(root)
    root.mainloop()
```

## ۷. تحلیل معماری پروژه

### چرا `get_filtered_students()` جداست؟

این تابع فقط مشخص می‌کند کدام دانشجوها با عبارت جست‌وجو و شهر انتخاب‌شده مطابقت دارند. این کار باعث می‌شود منطق فیلتر در یک نقطه متمرکز باشد و هم نمایش جدول و هم مرتب‌سازی بتوانند از آن استفاده کنند.

### چرا `refresh_table()` داده‌ها را دوباره می‌سازد؟

جدول نمایشی یک تصویر از داده‌های فعلی است. هر بار جست‌وجو یا فیلتر تغییر می‌کند، ردیف‌های قبلی حذف و ردیف‌های متناسب با شرایط جدید نمایش داده می‌شوند. فهرست `self.students` دست‌نخورده باقی می‌ماند، مگر آنکه کاربر رکوردی را حذف کند.

### مرتب‌سازی چه چیزی را تغییر می‌دهد؟

در این پروژه، مرتب‌سازی فقط ترتیب ردیف‌های نمایش‌داده‌شده را تغییر می‌دهد؛ ترتیب فهرست اصلی `self.students` عوض نمی‌شود. این انتخاب برای نمایش جدول مناسب است و باعث می‌شود ترتیب اصلی داده‌ها به‌طور ناخواسته تغییر نکند.

## ۸. تمرین‌های کلاسی

### تمرین ۱ — جست‌وجو بر اساس شناسه

جست‌وجو را طوری توسعه دهید که شناسه دانشجو نیز در عبارت جست‌وجو بررسی شود.

**پاسخ پیشنهادی:** در `get_filtered_students()` شرط `matches_query` را به این شکل گسترش دهید:

```python
matches_query = (
    query in student["name"].casefold()
    or query in student["city"].casefold()
    or query in str(student["id"])
)
```

### تمرین ۲ — فیلتر بر اساس بازه سنی

دو ورودی برای حداقل و حداکثر سن اضافه کنید و فقط دانشجوهایی را نشان دهید که سنشان در آن بازه قرار دارد.

**راهنمای پاسخ:**

1. مقدار ورودی‌ها را با `get()` بخوانید.
2. اگر ورودی خالی بود، برای آن محدودیت در نظر نگیرید.
3. ورودی غیرخالی را با `int()` تبدیل کنید و خطای `ValueError` را مدیریت کنید.
4. شرط سن را به فیلترهای موجود اضافه کنید.

### تمرین ۳ — نمایش تعداد نتایج

پس از جست‌وجو، تعداد نتایج نمایش‌داده‌شده را نشان دهید.

**پاسخ:** این قابلیت در `refresh_table()` با `self.status_label.config(...)` پیاده‌سازی شده است. توضیح دهید چرا تعداد نتایج با تعداد کل دانشجوها متفاوت است.

### تمرین ۴ — مرتب‌سازی نام

با کلیک روی سرستون Name، نام‌ها را یک بار صعودی و بار دیگر نزولی مرتب کنید.

**پاسخ:** تابع `sort_by()` وضعیت قبلی ستون را در `self.sort_state` نگه می‌دارد و جهت مرتب‌سازی را در هر کلیک تغییر می‌دهد.

## ۹. خطاهای رایج

- **مرتب‌سازی عدد به‌عنوان متن:** رشته‌های `"100"` و `"20"` از نظر لغوی ممکن است ترتیبی متفاوت از ترتیب عددی داشته باشند؛ ستون سن و شناسه را عددی مرتب کنید.
- **تغییر داده‌های اصلی هنگام فیلتر:** نتایج فیلتر را در یک فهرست جدا بسازید و `self.students` را فقط هنگام عملیات واقعی روی داده‌ها تغییر دهید.
- **اشتباه در `lambda`:** برای نگهداری مقدار هر ستون از `lambda col=column: ...` استفاده کنید.
- **تکراری بودن `iid`:** شناسه هر ردیف باید یکتا باشد.
- **فراموش کردن حالت بدون انتخاب:** قبل از خواندن یا حذف ردیف، `selection()` را بررسی کنید.
- **اشتباه گرفتن فیلتر با حذف:** فیلتر فقط نمایش را محدود می‌کند؛ نباید داده‌های نامنطبق را از فهرست اصلی پاک کند.

## ۱۰. جمع‌بندی

در این جلسه جدول را از یک نمایش ساده به رابطی کاربردی‌تر تبدیل کردیم:

- جست‌وجوی متن در نام و شهر
- فیلتر بر اساس شهر
- مرتب‌سازی صعودی و نزولی با کلیک روی سرستون
- نمایش تعداد نتایج
- انتخاب و حذف رکورد
- جداسازی داده‌های اصلی از جدول نمایشی

## ۱۱. چک‌لیست مدرس

- [ ] تفاوت داده اصلی و داده نمایشی را توضیح دادم.
- [ ] جست‌وجوی زنده را اجرا کردم.
- [ ] فیلتر شهر را نشان دادم.
- [ ] مرتب‌سازی عددی و متنی را مقایسه کردم.
- [ ] حالت بدون انتخاب را بررسی کردم.
- [ ] تأکید کردم که اطلاعات هنوز در حافظه‌اند و دائمی نیستند.

## جلسه بعد

در جلسه ۱۳ می‌توانیم مدیریت داده‌ها را به SQLite وصل کنیم تا اطلاعات پس از بسته شدن برنامه باقی بمانند. پیش از آن، دانشجو باید بتواند جدول، فیلتر، مرتب‌سازی و عملیات پایه روی رکوردها را توضیح دهد.
