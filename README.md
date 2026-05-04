import tkinter as tk
from tkinter import ttk, messagebox
import json
from datetime import datetime

class ExpenseTracker:
    def __init__(self, root):
        self.root = root
        self.root.title("Expense Tracker")
        self.data = []

        # Поля для ввода
        self.frame_input = ttk.Frame(root)
        self.frame_input.pack(padx=10, pady=10)

        ttk.Label(self.frame_input, text="Сумма:").grid(row=0, column=0)
        self.entry_sum = ttk.Entry(self.frame_input)
        self.entry_sum.grid(row=0, column=1)

        ttk.Label(self.frame_input, text="Категория:").grid(row=0, column=2)
        self.entry_category = ttk.Entry(self.frame_input)
        self.entry_category.grid(row=0, column=3)

        ttk.Label(self.frame_input, text="Дата (дд.мм.гггг):").grid(row=0, column=4)
        self.entry_date = ttk.Entry(self.frame_input)
        self.entry_date.grid(row=0, column=5)

        # Кнопка добавления
        self.btn_add = ttk.Button(self.frame_input, text="Добавить расход", command=self.add_expense)
        self.btn_add.grid(row=0, column=6, padx=5)

        # Таблица
        self.tree = ttk.Treeview(root, columns=("sum", "category", "date"), show='headings')
        self.tree.heading("sum", text="Сумма")
        self.tree.heading("category", text="Категория")
        self.tree.heading("date", text="Дата")
        self.tree.pack(padx=10, pady=10, fill='both', expand=True)

        # Фильтр
        self.frame_filter = ttk.Frame(root)
        self.frame_filter.pack(padx=10, pady=5)

        ttk.Label(self.frame_filter, text="Фильтр по категории:").grid(row=0, column=0)
        self.filter_category = ttk.Entry(self.frame_filter)
        self.filter_category.grid(row=0, column=1)

        ttk.Label(self.frame_filter, text="Фильтр по дате:").grid(row=0, column=2)
        self.filter_date = ttk.Entry(self.frame_filter)
        self.filter_date.grid(row=0, column=3)

        self.btn_filter = ttk.Button(self.frame_filter, text="Фильтровать", command=self.apply_filter)
        self.btn_filter.grid(row=0, column=4, padx=5)

        self.btn_clear_filter = ttk.Button(self.frame_filter, text="Очистить фильтр", command=self.clear_filter)
        self.btn_clear_filter.grid(row=0, column=5, padx=5)

        # Сумма за период
        self.label_total = ttk.Label(root, text="Общая сумма: 0")
        self.label_total.pack(pady=5)

        # Кнопки сохранения/загрузки
        self.frame_save = ttk.Frame(root)
        self.frame_save.pack(pady=5)

        ttk.Button(self.frame_save, text="Сохранить в JSON", command=self.save_json).grid(row=0, column=0, padx=5)
        ttk.Button(self.frame_save, text="Загрузить из JSON", command=self.load_json).grid(row=0, column=1, padx=5)

        self.load_json()

    def add_expense(self):
        try:
            sum_value = float(self.entry_sum.get())
            if sum_value <= 0:
                raise ValueError
        except ValueError:
            messagebox.showerror("Ошибка", "Введите положительное число для суммы")
            return

        category = self.entry_category.get()
        date_str = self.entry_date.get()
        try:
            date_obj = datetime.strptime(date_str, "%d.%m.%Y")
        except ValueError:
            messagebox.showerror("Ошибка", "Введите дату в формате дд.мм.гггг")
            return

        record = {
            "sum": sum_value,
            "category": category,
            "date": date_str
        }
        self.data.append(record)
        self.tree.insert('', 'end', values=(sum_value, category, date_str))
        self.update_total()

        self.entry_sum.delete(0, tk.END)
        self.entry_category.delete(0, tk.END)
        self.entry_date.delete(0, tk.END)

    def update_total(self):
        total = sum(item['sum'] for item in self.data)
        self.label_total.config(text=f"Общая сумма: {total:.2f}")

    def save_json(self):
        with open("expenses.json", "w", encoding='utf-8') as f:
            json.dump(self.data, f, ensure_ascii=False, indent=4)

    def load_json(self):
        try:
            with open("expenses.json", "r", encoding='utf-8') as f:
                self.data = json.load(f)
            for item in self.data:
                self.tree.insert('', 'end', values=(item['sum'], item['category'], item['date']))
            self.update_total()
        except FileNotFoundError:
            pass

    def apply_filter(self):
        category_filter = self.filter_category.get()
        date_filter = self.filter_date.get()

        # Очистка таблицы
        for item in self.tree.get_children():
            self.tree.delete(item)

        # Фильтрация
        filtered_data = self.data
        if category_filter:
            filtered_data = [d for d in filtered_data if d['category'] == category_filter]
        if date_filter:
            filtered_data = [d for d in filtered_data if d['date'] == date_filter]

        for item in filtered_data:
            self.tree.insert('', 'end', values=(item['sum'], item['category'], item['date']))
        self.update_total()

    def clear_filter(self):
        self.filter_category.delete(0, tk.END)
        self.filter_date.delete(0, tk.END)
        # Перезагрузка всех данных
        for item in self.tree.get_children():
            self.tree.delete(item)
        for item in self.data:
            self.tree.insert('', 'end', values=(item['sum'], item['category'], item['date']))
        self.update_total()

if __name__ == "__main__":
    root = tk.Tk()
    app = ExpenseTracker(root)
    root.mainloop()
