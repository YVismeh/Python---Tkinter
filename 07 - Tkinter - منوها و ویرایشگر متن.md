---
title: "Tkinter - جلسه ۷ - منوها و ویرایشگر متن"
aliases:
  - Tkinter Session 7
  - Menu و Text و File Dialog
tags:
  - Python
  - Tkinter
  - GUI
  - آموزش
level: intermediate
status: ready
---

# جلسه ۷ — منوها، فایل‌ها و ویرایشگر متن ساده

## هدف جلسه

در این جلسه یاد می‌گیریم چگونه یک برنامه دسکتاپ بسازیم که کاربر بتواند از طریق منو، فایل متنی جدید ایجاد کند، فایل قبلی را باز کند، متن را ذخیره کند و از برنامه خارج شود.

در پایان جلسه، یک **ویرایشگر متن ساده** خواهیم داشت.

## پیش‌نیازها

- ساخت پنجره با `Tk`
- کار با `Button`، `Entry` و `Frame`
- آشنایی با رویدادها و تابع‌ها
- آشنایی مقدماتی با `try/except`

## ۱. آشنایی با ویجت `Text`

ویجت `Entry` برای دریافت یک خط متن مناسب است؛ اما `Text` برای متن‌های چندخطی به کار می‌رود.

```python
import tkinter as tk

root = tk.Tk()
root.title("Text Widget Example")
root.geometry("500x350")

text_box = tk.Text(root, width=50, height=15, wrap="word")
text_box.pack(fill="both", expand=True, padx=10, pady=10)

root.mainloop()
```

نکته‌های مهم:

- `height` معمولاً ارتفاع را بر حسب تعداد خط مشخص می‌کند.
- `wrap="word"` باعث می‌شود متن در مرز کلمات به خط بعد منتقل شود.
- `fill="both"` و `expand=True` به ویجت اجازه می‌دهند فضای موجود را بگیرد.

### دریافت متن از `Text`

برای دریافت همه متن، از اندیس `"1.0"` تا `"end"` استفاده می‌کنیم:

```python
content = text_box.get("1.0", "end")
```

- `"1.0"` یعنی خط اول، ستون صفر.
- `"end"` انتهای محتوای ویجت را مشخص می‌کند.
- Tkinter معمولاً یک خط جدید پایانی به متن `Text` اضافه می‌کند؛ برای حذف آن می‌توانیم از `end-1c` استفاده کنیم:

```python
content = text_box.get("1.0", "end-1c")
```

### پاک کردن و جایگزینی متن

```python
text_box.delete("1.0", "end")
text_box.insert("1.0", "New text")
```

ابتدا متن قبلی پاک می‌شود و سپس متن جدید درج می‌شود.

## ۲. ساخت منو با `Menu`

منوها راهی استاندارد برای قرار دادن دستورهایی مانند New، Open، Save و Exit هستند.

```python
import tkinter as tk

root = tk.Tk()
root.title("Menu Example")
root.geometry("500x350")

menu_bar = tk.Menu(root)

file_menu = tk.Menu(menu_bar, tearoff=0)
file_menu.add_command(label="New", command=lambda: print("New file"))
file_menu.add_command(label="Open", command=lambda: print("Open file"))
file_menu.add_separator()
file_menu.add_command(label="Exit", command=root.destroy)

menu_bar.add_cascade(label="File", menu=file_menu)
root.config(menu=menu_bar)

root.mainloop()
```

مفاهیم مهم:

- `Menu(root)` نوار منو را می‌سازد.
- `Menu(menu_bar, tearoff=0)` منوی کشویی را می‌سازد.
- `add_command()` یک گزینه قابل انتخاب اضافه می‌کند.
- `add_separator()` خط جداکننده اضافه می‌کند.
- `add_cascade()` منوی کشویی را به نوار اصلی متصل می‌کند.
- `command=` تابعی را مشخص می‌کند که پس از انتخاب گزینه اجرا می‌شود.

در برنامه‌های واقعی، به‌جای `print()`، تابع‌هایی برای انجام عملیات تعریف می‌کنیم.

## ۳. انتخاب فایل با `filedialog`

ماژول `filedialog` پنجره استاندارد انتخاب فایل را باز می‌کند.

### باز کردن پنجره انتخاب فایل

```python
from tkinter import filedialog

file_path = filedialog.askopenfilename(
    title="Open a text file",
    filetypes=[("Text files", "*.txt"), ("All files", "*.*")]
)

if file_path:
    print(file_path)
```

اگر کاربر پنجره را ببندد یا Cancel را انتخاب کند، مقدار بازگشتی رشته خالی است. بنابراین قبل از باز کردن فایل باید آن را بررسی کنیم.

### انتخاب مسیر ذخیره

```python
file_path = filedialog.asksaveasfilename(
    title="Save file",
    defaultextension=".txt",
    filetypes=[("Text files", "*.txt"), ("All files", "*.*")]
)
```

`asksaveasfilename()` مسیر را انتخاب می‌کند؛ خودش محتوای فایل را ذخیره نمی‌کند. ذخیره واقعی با `open()` انجام می‌شود.

## ۴. خواندن و نوشتن فایل متنی

برای خواندن:

```python
with open(file_path, "r", encoding="utf-8") as file:
    content = file.read()
```

برای نوشتن:

```python
with open(file_path, "w", encoding="utf-8") as file:
    file.write(content)
```

نکته‌ها:

- `with open(...)` پس از پایان کار فایل را به‌درستی می‌بندد.
- `encoding="utf-8"` برای ذخیره و خواندن متن فارسی مناسب است.
- حالت `"w"` محتوای قبلی فایل را جایگزین می‌کند؛ پس باید آگاهانه از آن استفاده کنیم.
- خطاهایی مانند نداشتن دسترسی یا مشکل در خواندن فایل ممکن است رخ دهند؛ در برنامه از `try/except` استفاده می‌کنیم.

## ۵. پروژه کامل: ویرایشگر متن ساده

امکانات پروژه:

- منوی File
- ایجاد فایل جدید
- باز کردن فایل متنی
- ذخیره فایل
- ذخیره با نام جدید
- خروج از برنامه
- هشدار هنگام بستن برنامه در صورت وجود تغییرات ذخیره‌نشده

کد زیر را در فایلی مانند `text_editor.py` ذخیره و اجرا کنید. تمام متن‌های داخل کد انگلیسی هستند تا بتوانید کد را مستقیماً در محیط برنامه‌نویسی اجرا کنید.

```python
import tkinter as tk
from tkinter import filedialog, messagebox
from pathlib import Path


class TextEditor:
    def __init__(self, root):
        self.root = root
        self.root.title("Simple Text Editor")
        self.root.geometry("800x550")

        self.current_file = None
        self.is_modified = False

        self.text_box = tk.Text(
            root,
            wrap="word",
            undo=True,
            font=("Arial", 12)
        )
        self.text_box.pack(fill="both", expand=True)

        self.status_label = tk.Label(
            root,
            text="Ready",
            anchor="w"
        )
        self.status_label.pack(fill="x", padx=5, pady=3)

        self._create_menu()

        self.text_box.bind("<<Modified>>", self._on_text_modified)
        self.root.protocol("WM_DELETE_WINDOW", self.exit_editor)

        self._update_title()

    def _create_menu(self):
        menu_bar = tk.Menu(self.root)

        file_menu = tk.Menu(menu_bar, tearoff=0)
        file_menu.add_command(label="New", command=self.new_file)
        file_menu.add_command(label="Open...", command=self.open_file)
        file_menu.add_separator()
        file_menu.add_command(label="Save", command=self.save_file)
        file_menu.add_command(label="Save As...", command=self.save_file_as)
        file_menu.add_separator()
        file_menu.add_command(label="Exit", command=self.exit_editor)

        menu_bar.add_cascade(label="File", menu=file_menu)
        self.root.config(menu=menu_bar)

    def _on_text_modified(self, event=None):
        if self.text_box.edit_modified():
            self.is_modified = True
            self.status_label.config(text="Modified")
            self._update_title()
            self.text_box.edit_modified(False)

    def _update_title(self):
        if self.current_file:
            filename = Path(self.current_file).name
        else:
            filename = "Untitled"

        marker = " *" if self.is_modified else ""
        self.root.title(f"{filename}{marker} - Simple Text Editor")

    def _confirm_discard_changes(self):
        if not self.is_modified:
            return True

        answer = messagebox.askyesnocancel(
            "Unsaved Changes",
            "Do you want to save your changes?"
        )

        if answer is None:
            return False

        if answer is True:
            return self.save_file()

        return True

    def new_file(self):
        if not self._confirm_discard_changes():
            return

        self.text_box.delete("1.0", "end")
        self.current_file = None
        self.is_modified = False
        self.text_box.edit_modified(False)
        self.status_label.config(text="New file")
        self._update_title()

    def open_file(self):
        if not self._confirm_discard_changes():
            return

        file_path = filedialog.askopenfilename(
            title="Open text file",
            filetypes=[
                ("Text files", "*.txt"),
                ("All files", "*.*")
            ]
        )

        if not file_path:
            return

        try:
            with open(file_path, "r", encoding="utf-8") as file:
                content = file.read()
        except (OSError, UnicodeError) as error:
            messagebox.showerror("Open Error", str(error))
            return

        self.text_box.delete("1.0", "end")
        self.text_box.insert("1.0", content)
        self.current_file = file_path
        self.is_modified = False
        self.text_box.edit_modified(False)
        self.status_label.config(text="File opened")
        self._update_title()

    def save_file(self):
        if self.current_file is None:
            return self.save_file_as()

        return self._write_file(self.current_file)

    def save_file_as(self):
        file_path = filedialog.asksaveasfilename(
            title="Save text file",
            defaultextension=".txt",
            filetypes=[
                ("Text files", "*.txt"),
                ("All files", "*.*")
            ]
        )

        if not file_path:
            return False

        return self._write_file(file_path)

    def _write_file(self, file_path):
        content = self.text_box.get("1.0", "end-1c")

        try:
            with open(file_path, "w", encoding="utf-8") as file:
                file.write(content)
        except OSError as error:
            messagebox.showerror("Save Error", str(error))
            return False

        self.current_file = file_path
        self.is_modified = False
        self.text_box.edit_modified(False)
        self.status_label.config(text="Saved")
        self._update_title()
        return True

    def exit_editor(self):
        if self._confirm_discard_changes():
            self.root.destroy()


if __name__ == "__main__":
    root = tk.Tk()
    app = TextEditor(root)
    root.mainloop()
```

## ۶. بررسی بخش‌های مهم پروژه

### چرا از کلاس استفاده کرده‌ایم؟

برنامه چند وضعیت دارد: مسیر فایل فعلی، تغییرات ذخیره‌نشده و خود ویجت متن. قرار دادن این موارد در یک کلاس باعث می‌شود مدیریت برنامه منظم‌تر شود.

### چرا `current_file` مقدار `None` دارد؟

در شروع، هنوز فایلی باز نشده است. وقتی کاربر Save را انتخاب کند، برنامه متوجه می‌شود که باید ابتدا مسیر فایل را از او بگیرد.

### تفاوت Save و Save As چیست؟

- **Save:** فایل را در مسیر فعلی ذخیره می‌کند.
- **Save As:** مسیر جدیدی از کاربر می‌گیرد و فایل را در آن مسیر ذخیره می‌کند.

### چرا `return self.save_file()` مهم است؟

اگر کاربر هنگام ایجاد فایل جدید یا خروج، گزینه Save را انتخاب کند اما پنجره ذخیره را Cancel کند، تابع `save_file()` مقدار `False` برمی‌گرداند. در این حالت عملیات ایجاد فایل جدید یا خروج متوقف می‌شود تا متن کاربر ناخواسته از بین نرود.

### رویداد `<<Modified>>`

این رویداد به تغییر وضعیت ویرایش ویجت `Text` مربوط است. پس از بررسی آن، با `edit_modified(False)` علامت تغییر را پاک می‌کنیم تا رویدادهای بعدی قابل تشخیص باشند.

## ۷. تمرین‌های دانشجو

### تمرین ۱ — منوی Edit

یک منوی `Edit` اضافه کنید که گزینه‌های زیر را داشته باشد:

- Select All
- Clear

**پاسخ پیشنهادی:**

```python
def select_all(self, event=None):
    self.text_box.tag_add("sel", "1.0", "end-1c")
    self.text_box.mark_set("insert", "1.0")
    self.text_box.see("insert")
    return "break"

def clear_text(self):
    self.text_box.delete("1.0", "end")
```

برای اتصال این تابع‌ها به منو، داخل `_create_menu()` بنویسید:

```python
edit_menu = tk.Menu(menu_bar, tearoff=0)
edit_menu.add_command(label="Select All", command=self.select_all)
edit_menu.add_command(label="Clear", command=self.clear_text)
menu_bar.add_cascade(label="Edit", menu=edit_menu)
```

### تمرین ۲ — نمایش تعداد کلمات

تابعی بنویسید که تعداد کلمات متن را محاسبه کند.

**پاسخ:**

```python
def count_words(self):
    content = self.text_box.get("1.0", "end-1c")
    word_count = len(content.split())
    messagebox.showinfo("Word Count", f"Words: {word_count}")
```

این تابع را می‌توانید به گزینه‌ای در منوی Tools متصل کنید.

### تمرین ۳ — پاک کردن متن با تأیید

قبل از پاک کردن متن از کاربر تأیید بگیرید.

**پاسخ:**

```python
def clear_text(self):
    confirmed = messagebox.askyesno(
        "Clear Text",
        "Are you sure you want to clear the text?"
    )

    if confirmed:
        self.text_box.delete("1.0", "end")
```

### تمرین ۴ — میانبر صفحه‌کلید

برای ذخیره فایل، میانبر `Ctrl+S` اضافه کنید.

**پاسخ:**

```python
self.root.bind("<Control-s>", self._save_shortcut)

def _save_shortcut(self, event=None):
    self.save_file()
    return "break"
```

این دو بخش را در کلاس `TextEditor` قرار دهید؛ خط `bind` را داخل `__init__` اضافه کنید و تابع را در بدنه کلاس تعریف کنید.

## ۸. خطاهای رایج

- **استفاده از `Entry` برای متن چندخطی:** برای متن طولانی از `Text` استفاده کنید.
- **فراموش کردن بررسی مسیر خالی:** ممکن است کاربر پنجره انتخاب فایل را لغو کند.
- **استفاده نکردن از UTF-8:** ممکن است متن فارسی به‌درستی خوانده یا ذخیره نشود.
- **فراموش کردن `end-1c`:** ممکن است یک خط جدید اضافه در انتهای متن ذخیره شود.
- **نوشتن مستقیم فایل بدون مدیریت خطا:** عملیات فایل ممکن است به دلیل دسترسی یا مشکلات دیگر شکست بخورد.
- **ذخیره با حالت `w` بدون بررسی مسیر:** حالت `w` محتوای فایل مقصد را جایگزین می‌کند.

## ۹. جمع‌بندی جلسه

- `Text` برای متن چندخطی است.
- `Menu` برای ساخت منوهای برنامه استفاده می‌شود.
- `filedialog` پنجره انتخاب مسیر فایل را باز می‌کند.
- `open()` برای خواندن و نوشتن فایل به کار می‌رود.
- `messagebox` برای نمایش پیام و گرفتن تأیید استفاده می‌شود.
- `WM_DELETE_WINDOW` اجازه می‌دهد پیش از بسته شدن پنجره، تغییرات ذخیره‌نشده بررسی شوند.
- جدا کردن عملیات به تابع‌های کوچک، نگهداری و توسعه برنامه را آسان‌تر می‌کند.

## ۱۰. چک‌لیست مدرس

- [ ] تفاوت `Entry` و `Text` را توضیح دادم.
- [ ] منوی اصلی و منوی کشویی را ساختم.
- [ ] باز کردن و ذخیره فایل را اجرا کردم.
- [ ] تفاوت Save و Save As را توضیح دادم.
- [ ] رفتار برنامه هنگام لغو پنجره ذخیره را آزمایش کردم.
- [ ] دانشجو حداقل دو تمرین را انجام داد.

## جلسه بعد

در جلسه ۸ می‌توانیم ذخیره‌سازی ساخت‌یافته با JSON را بررسی کنیم و یک برنامه فهرست کارها (To-Do List) بسازیم تا اطلاعات پس از بسته شدن برنامه نیز باقی بمانند.
