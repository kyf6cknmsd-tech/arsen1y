# arsen1y
# Expense Tracker

Автор: Арсений Фефелов

## Описание
Программа предназначена для отслеживания личных расходов. Позволяет добавлять расходы, фильтровать их по категории и дате, а также подсчитывать сумму за выбранный период. Данные сохраняются в JSON-файле.

## Как использовать
1. Запустите `main.py`.
2. Введите сумму, выберите категорию и укажите дату (формат ГГГГ-ММ-ДД).
3. Нажмите «Добавить расход».
4. Для фильтрации выберите параметры и нажмите «Применить фильтр».
5. Для подсчёта суммы за выбранный фильтр нажмите кнопку «Посчитать сумму за фильтрованный период».
import tkinter as tk
from tkinter import ttk, messagebox
import json
from datetime import datetime

# Файл для хранения данных
DATA_FILE = 'expenses.json'


# Загрузка данных из файла
def load_data():
    try:
        with open(DATA_FILE, 'r') as f:
            return json.load(f)
    except (FileNotFoundError, json.JSONDecodeError):
        return []

# Сохранение данных в файл
def save_data(data):
    with open(DATA_FILE, 'w') as f:
        json.dump(data, f, indent=4)


# Основное окно
root = tk.Tk()
root.title("Expense Tracker")
root.geometry("800x600")

# Данные
expenses = load_data()

# --- Создание элементов формы ---
# Поля ввода
frame_form = tk.Frame(root)
frame_form.pack(pady=10)

tk.Label(frame_form, text="Сумма").grid(row=0, column=0, padx=5)
entry_amount = tk.Entry(frame_form)
entry_amount.grid(row=0, column=1, padx=5)

tk.Label(frame_form, text="Категория").grid(row=0, column=2, padx=5)
category_var = tk.StringVar()
category_menu = ttk.Combobox(frame_form, textvariable=category_var)
category_menu['values'] = ('Еда', 'Транспорт', 'Развлечения', 'Другое')
category_menu.current(0)
category_menu.grid(row=0, column=3, padx=5)

tk.Label(frame_form, text="Дата (ГГГГ-ММ-ДД)").grid(row=0, column=4, padx=5)
entry_date = tk.Entry(frame_form)
entry_date.insert(0, datetime.now().strftime('%Y-%m-%d'))
entry_date.grid(row=0, column=5, padx=5)

# Кнопка для добавления расхода
def add_expense():
    amount_text = entry_amount.get()
    category = category_var.get()
    date_text = entry_date.get()
    # Валидация
    try:
        amount = float(amount_text)
        if amount <= 0:
            raise ValueError
    except:
        messagebox.showerror("Ошибка", "Введите положительное число для суммы.")
        return
    # Проверка формата даты
    try:
        datetime.strptime(date_text, '%Y-%m-%d')
    except:
        messagebox.showerror("Ошибка", "Неправильный формат даты. Используйте ГГГГ-ММ-ДД.")
        return
    # Добавление записи
    expense = {
        'amount': amount,
        'category': category,
        'date': date_text
    }
    expenses.append(expense)
    save_data(expenses)
    refresh_tree()
    clear_fields()

def clear_fields():
    entry_amount.delete(0, tk.END)
    entry_date.delete(0, tk.END)
    entry_date.insert(0, datetime.now().strftime('%Y-%m-%d'))

btn_add = tk.Button(frame_form, text="Добавить расход", command=add_expense)
btn_add.grid(row=0, column=6, padx=5)

# --- Таблица для отображения расходов ---
columns = ('amount', 'category', 'date')
tree = ttk.Treeview(root, columns=columns, show='headings', height=15)

for col in columns:
    tree.heading(col, text=col.capitalize())

tree.pack(pady=10, fill='both', expand=True)

def refresh_tree():
    for row in tree.get_children():
        tree.delete(row)
    for expense in expenses:
        tree.insert('', tk.END, values=(expense['amount'], expense['category'], expense['date']))

refresh_tree()

# --- Фильтрация ---
filter_frame = tk.Frame(root)
filter_frame.pack(pady=10)

tk.Label(filter_frame, text="Фильтр по категории").grid(row=0, column=0, padx=5)
filter_category_var = tk.StringVar(value='Все')
category_options = ('Все',) + ('Еда', 'Транспорт', 'Развлечения', 'Другое')
filter_category_menu = ttk.Combobox(filter_frame, textvariable=filter_category_var, values=category_options)
filter_category_menu.current(0)
filter_category_menu.grid(row=0, column=1, padx=5)

tk.Label(filter_frame, text="Фильтр по дате").grid(row=0, column=2, padx=5)
filter_date_entry = tk.Entry(filter_frame)
filter_date_entry.insert(0, 'Все')
filter_date_entry.grid(row=0, column=3, padx=5)

def apply_filter():
    cat_filter = filter_category_var.get()
    date_filter = filter_date_entry.get()
    filtered = []

    for exp in expenses:
        if (cat_filter == 'Все' or exp['category'] == cat_filter) and \
           (date_filter == 'Все' or exp['date'] == date_filter):
            filtered.append(exp)
    # Обновление таблицы
    for row in tree.get_children():
        tree.delete(row)
    for expense in filtered:
        tree.insert('', tk.END, values=(expense['amount'], expense['category'], expense['date']))

btn_filter = tk.Button(filter_frame, text="Применить фильтр", command=apply_filter)
btn_filter.grid(row=0, column=4, padx=5)

# --- Итоговая сумма за выбранный период ---
def calculate_total():
    total = 0
    cat_filter = filter_category_var.get()
    date_filter = filter_date_entry.get()
    for exp in expenses:
        if (cat_filter == 'Все' or exp['category'] == cat_filter) and \
           (date_filter == 'Все' or exp['date'] == date_filter):
            total += exp['amount']
    messagebox.showinfo("Общая сумма", f"Общая сумма расходов: {total:.2f}")

total_btn = tk.Button(root, text="Посчитать сумму за фильтрованный период", command=calculate_total)
total_btn.pack(pady=10)

# ----- Запуск приложения -----
root.mainloop()



