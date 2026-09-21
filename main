import tkinter as tk
from tkinter import ttk, messagebox
import csv
import os
from datetime import datetime
from collections import defaultdict
import matplotlib.pyplot as plt
from openpyxl import Workbook

# 
# FILE & DATA SETUP
# 

BASE_DIR = os.path.dirname(os.path.abspath(__file__))
FILENAME = os.path.join(BASE_DIR, "expenses.csv")
BUDGET_FILE = os.path.join(BASE_DIR, "budget.txt")

def initialize_files():
    if not os.path.exists(FILENAME):
        with open(FILENAME, "w", newline="", encoding="utf-8") as file:
            writer = csv.writer(file)
            writer.writerow(["Date", "Category", "Amount"])
            
    if not os.path.exists(BUDGET_FILE):
        with open(BUDGET_FILE, "w", encoding="utf-8") as file:
            file.write("15000.0")

def get_budget_limit():
    try:
        with open(BUDGET_FILE, "r", encoding="utf-8") as file:
            return float(file.read().strip())
    except:
        return 15000.0

def save_budget_limit(new_limit):
    with open(BUDGET_FILE, "w", encoding="utf-8") as file:
        file.write(str(new_limit))

initialize_files()


# 
# CORE APPLICATION LOGIC
# 

def add_expense():
    category = category_entry.get().strip()
    amount_str = amount_entry.get().strip()

    if not category:
        messagebox.showerror("Error", "Category field cannot be blank.")
        return

    try:
        amount = float(amount_str)
        if amount <= 0:
            raise ValueError
    except ValueError:
        messagebox.showerror("Error", "Please input a positive numeric value.")
        return

    current_total = calculate_total_expenses()
    budget_limit = get_budget_limit()
    
    if current_total + amount > budget_limit:
        proceed = messagebox.askyesno("Budget Alert", f"Warning: Adding this expense (₹{amount:,.2f}) will push you over your monthly limit of ₹{budget_limit:,.2f}!\n\nDo you still want to log it?")
        if not proceed:
            return

    with open(FILENAME, "a", newline="", encoding="utf-8") as file:
        writer = csv.writer(file)
        writer.writerow([
            datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
            category,
            amount
        ])

    messagebox.showinfo("Success", "Expense transaction logged.")
    category_entry.delete(0, tk.END)
    amount_entry.delete(0, tk.END)
    
    refresh_all_views()


def calculate_total_expenses():
    total = 0.0
    if os.path.exists(FILENAME):
        with open(FILENAME, "r", newline="", encoding="utf-8") as file:
            reader = csv.DictReader(file)
            for row in reader:
                try:
                    total += float(row["Amount"])
                except ValueError:
                    pass
    return total


def refresh_all_views():
    current_total = calculate_total_expenses()
    budget_limit = get_budget_limit()
    remaining = budget_limit - current_total
    
    # Update UI Headers
    budget_lbl.config(text=f"Monthly Limit: ₹{budget_limit:,.2f}")
    spent_lbl.config(text=f"Total Logged: ₹{current_total:,.2f}")
    remaining_lbl.config(text=f"Remaining: ₹{remaining:,.2f}")
    
    # Use yellow highlighting for warning indicator if remaining balance is low or negative
    if remaining < 0:
        remaining_lbl.config(bg="#EAB308", fg="#000000") # Darker, readable text-safe yellow accent
    else:
        remaining_lbl.config(bg="#FFFFFF", fg="#000000")

    # Reload Table
    for item in tree.get_children():
        tree.delete(item)

    with open(FILENAME, "r", newline="", encoding="utf-8") as file:
        reader = csv.DictReader(file)
        for idx, row in enumerate(reader):
            amt = float(row["Amount"])
            tree.insert("", tk.END, iid=str(idx), values=(row["Date"], row["Category"], f"{amt:.2f}"))


def update_budget_setting():
    new_limit_str = budget_setup_entry.get().strip()
    try:
        new_limit = float(new_limit_str)
        if new_limit < 0:
            raise ValueError
        save_budget_limit(new_limit)
        messagebox.showinfo("Success", "Monthly budget tracking ceiling updated.")
        budget_setup_entry.delete(0, tk.END)
        refresh_all_views()
    except ValueError:
        messagebox.showerror("Error", "Please enter a valid positive number for your limit.")


def delete_selected_expense():
    selected_item = tree.selection()
    if not selected_item:
        messagebox.showwarning("Selection Required", "Please tap a specific row from the log list below to delete.")
        return

    confirm = messagebox.askyesno("Confirm Action", "Permanently purge this item line from tracking storage history?")
    if not confirm:
        return

    target_index = int(selected_item[0])

    rows = []
    with open(FILENAME, "r", newline="", encoding="utf-8") as file:
        reader = csv.reader(file)
        rows = list(reader)

    if len(rows) > (target_index + 1):
        rows.pop(target_index + 1)

    with open(FILENAME, "w", newline="", encoding="utf-8") as file:
        writer = csv.writer(file)
        writer.writerows(rows)

    messagebox.showinfo("Updated", "Record item deleted.")
    refresh_all_views()


def show_summary():
    summary = defaultdict(float)
    with open(FILENAME, "r", newline="", encoding="utf-8") as file:
        reader = csv.DictReader(file)
        for row in reader:
            summary[row["Category"]] += float(row["Amount"])

    summary_text.config(state=tk.NORMAL)
    summary_text.delete("1.0", tk.END)
    
    if not summary:
        summary_text.insert(tk.END, "No recorded structural data historical profiles found.")
    else:
        summary_text.insert(tk.END, f"{'CATEGORY':<25} | {'AGGREGATE TOTAL'}\n")
        summary_text.insert(tk.END, "─" * 45 + "\n")
        for category, amount in summary.items():
            summary_text.insert(tk.END, f"{category.upper():<25} | ₹{amount:,.2f}\n")
            
    summary_text.config(state=tk.DISABLED)


def show_pie_chart():
    summary = defaultdict(float)
    with open(FILENAME, "r", newline="", encoding="utf-8") as file:
        reader = csv.DictReader(file)
        for row in reader:
            summary[row["Category"]] += float(row["Amount"])

    if not summary:
        messagebox.showinfo("Empty Context", "No graphical distributions to draw.")
        return

    # Pie chart styled to match the light theme with yellow accents
    plt.style.use('default')
    fig, ax = plt.subplots(figsize=(6, 6))
    fig.patch.set_facecolor('#FFFFFF')
    ax.set_facecolor('#FFFFFF')
    
    # White, Grey, and Yellow custom slice colors
    colors = ['#E6E6E6', '#CCCCCC', '#FEF08A', '#FEF9C3', '#A3A3A3']
    ax.pie(list(summary.values()), labels=list(summary.keys()), autopct="%1.1f%%", startangle=140, colors=colors, textprops={'color':"black"}, wedgeprops={'edgecolor':'black'})
    plt.title("Expense Volume Split Profiles", color='black', pad=20, fontsize=14, weight='bold')
    plt.tight_layout()
    plt.show()


def perform_search():
    for item in search_tree.get_children():
        search_tree.delete(item)

    target = search_entry.get().strip().lower()
    if not target:
        messagebox.showwarning("Warning", "Enter a query identity string.")
        return

    found = False
    with open(FILENAME, "r", newline="", encoding="utf-8") as file:
        reader = csv.DictReader(file)
        for row in reader:
            if target in row["Category"].lower():
                search_tree.insert("", tk.END, values=(row["Date"], row["Category"], row["Amount"]))
                found = True

    if not found:
        messagebox.showinfo("System Trace", "Zero category intersections established.")


def export_to_excel():
    workbook = Workbook()
    sheet = workbook.active
    sheet.title = "Ledger Records"
    sheet.append(["Date", "Category", "Amount"])

    with open(FILENAME, "r", newline="", encoding="utf-8") as file:
        reader = csv.DictReader(file)
        for row in reader:
            sheet.append([row["Date"], row["Category"], float(row["Amount"])])

    output_file = os.path.join(BASE_DIR, "expenses_report.xlsx")
    workbook.save(output_file)
    messagebox.showinfo("Export Done", f"Saved successfully:\n{output_file}")


# 
# WHITE / GREY / YELLOW GUI
# 

root = tk.Tk()
root.title("Capital Ledger Engine")
root.geometry("850x650")
root.configure(bg="#FFFFFF") # Absolute White Base

# Strictly Defined Minimal Palette 
BG_WHITE = "#FFFFFF"
BG_LIGHT_GREY = "#E5E7EB"
BG_HOVER_GREY = "#D1D5DB"
TEXT_BLACK = "#000000"
ACCENT_YELLOW = "#FEF08A" # Soft theme-friendly yellow for alerts

style = ttk.Style()
style.theme_use("default")

# Universal Type and Container Setup
style.configure(".", bg=BG_WHITE, foreground=TEXT_BLACK, font=("Segoe UI", 10))
style.configure("TFrame", background=BG_WHITE)
style.configure("TLabelframe", background=BG_WHITE, bordercolor=BG_LIGHT_GREY, padding=15)
style.configure("TLabelframe.Label", background=BG_WHITE, foreground=TEXT_BLACK, font=("Segoe UI", 10, "bold"))

# Clean Minimalist Tab Notebook Control
style.configure("TNotebook", background=BG_WHITE, borderwidth=0)
style.configure("TNotebook.Tab", background=BG_LIGHT_GREY, foreground=TEXT_BLACK, borderwidth=1, bordercolor="#9CA3AF", padding=[15, 7], font=("Segoe UI", 10, "bold"))
style.map("TNotebook.Tab", background=[("selected", BG_WHITE)], foreground=[("selected", TEXT_BLACK)])

# Minimalist Form Fields
style.configure("TEntry", fieldbackground=BG_WHITE, foreground=TEXT_BLACK, borderwidth=1, bordercolor="#9CA3AF")

# Light Grey Styled Buttons
style.configure("TButton", background=BG_LIGHT_GREY, foreground=TEXT_BLACK, borderwidth=1, bordercolor="#9CA3AF", padding=[15, 8], font=("Segoe UI", 10, "bold"), anchor="center")
style.map("TButton", background=[("active", BG_HOVER_GREY)], foreground=[("active", TEXT_BLACK)])

# Danger Button using a yellow-highlight override layout option
style.configure("Danger.TButton", background=BG_LIGHT_GREY, foreground=TEXT_BLACK, borderwidth=1, bordercolor="#9CA3AF")
style.map("Danger.TButton", background=[("active", ACCENT_YELLOW)])

# Stark High-Contrast Data Tables
style.configure("Treeview", background=BG_WHITE, fieldbackground=BG_WHITE, foreground=TEXT_BLACK, rowheight=28, borderwidth=1, bordercolor=BG_LIGHT_GREY, font=("Segoe UI", 10))
style.configure("Treeview.Heading", background=BG_LIGHT_GREY, foreground=TEXT_BLACK, borderwidth=1, bordercolor="#9CA3AF", padding=5, font=("Segoe UI", 10, "bold"))
style.map("Treeview", background=[("selected", BG_LIGHT_GREY)], foreground=[("selected", TEXT_BLACK)])

# UPPER TOP CAP: METRIC HEALTH DISPLAY
health_ribbon = tk.Frame(root, bg=BG_WHITE, borderwidth=1, relief="solid", highlightthickness=0)
health_ribbon.pack(fill=tk.X, padx=15, pady=10)

budget_lbl = tk.Label(health_ribbon, text="Monthly Limit: ₹0.00", bg=BG_WHITE, fg=TEXT_BLACK, font=("Segoe UI", 11, "bold"))
budget_lbl.pack(side=tk.LEFT, padx=15, pady=10)

spent_lbl = tk.Label(health_ribbon, text="Total Logged: ₹0.00", bg=BG_WHITE, fg=TEXT_BLACK, font=("Segoe UI", 11, "bold"))
spent_lbl.pack(side=tk.LEFT, padx=15, pady=10)

remaining_lbl = tk.Label(health_ribbon, text="Remaining: ₹0.00", bg=BG_WHITE, fg=TEXT_BLACK, font=("Segoe UI", 11, "bold"))
remaining_lbl.pack(side=tk.LEFT, padx=15, pady=10)


# NOTEBOOK INTERFACE
notebook = ttk.Notebook(root)
notebook.pack(fill=tk.BOTH, expand=True, padx=15, pady=5)

tab1 = ttk.Frame(notebook)
tab2 = ttk.Frame(notebook)
tab3 = ttk.Frame(notebook)
tab4 = ttk.Frame(notebook)

notebook.add(tab1, text="  Add Expense  ")
notebook.add(tab2, text="  View Log  ")
notebook.add(tab3, text="  Analytics  ")
notebook.add(tab4, text="  Search & Adjust Limit  ")


# TAB 1: ADD EXPENSE
add_container = ttk.Frame(tab1, padding=20)
add_container.pack(fill=tk.BOTH, expand=True)

form_box = ttk.LabelFrame(add_container, text=" Log New Expense Entry ")
form_box.pack(pady=20, padx=40, fill=tk.X)

ttk.Label(form_box, text="Category Designation:").pack(anchor="w", pady=(10, 2))
category_entry = ttk.Entry(form_box, width=45)
category_entry.pack(fill=tk.X, pady=5)

ttk.Label(form_box, text="Transaction Cost (₹):").pack(anchor="w", pady=(10, 2))
amount_entry = ttk.Entry(form_box, width=45)
amount_entry.pack(fill=tk.X, pady=5)

ttk.Button(form_box, text="Save Expense Record", command=add_expense).pack(pady=20)


# TAB 2: VIEW LOG & MANAGEMENT
log_container = ttk.Frame(tab2, padding=15)
log_container.pack(fill=tk.BOTH, expand=True)

tree_scroll_wrapper = ttk.Frame(log_container)
tree_scroll_wrapper.pack(fill=tk.BOTH, expand=True)

tree = ttk.Treeview(tree_scroll_wrapper, columns=("Date", "Category", "Amount"), show="headings")
tree.heading("Date", text="Timestamp")
tree.heading("Category", text="Expense Category")
tree.heading("Amount", text="Amount (₹)")
tree.column("Date", width=180, anchor="center")
tree.column("Category", width=260, anchor="w")
tree.column("Amount", width=120, anchor="e")

scroll_y = ttk.Scrollbar(tree_scroll_wrapper, orient=tk.VERTICAL, command=tree.yview)
tree.configure(yscrollcommand=scroll_y.set)
tree.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)
scroll_y.pack(side=tk.RIGHT, fill=tk.Y)

action_tray = ttk.Frame(log_container, padding=(0, 10, 0, 0))
action_tray.pack(fill=tk.X)

ttk.Button(action_tray, text="Export Array (.xlsx)", command=export_to_excel).pack(side=tk.LEFT, padx=5)
ttk.Button(action_tray, text="Delete Selected Row", style="Danger.TButton", command=delete_selected_expense).pack(side=tk.RIGHT, padx=5)


# TAB 3: ANALYTICS
analytics_container = ttk.Frame(tab3, padding=15)
analytics_container.pack(fill=tk.BOTH, expand=True)

control_strip = ttk.Frame(analytics_container)
control_strip.pack(fill=tk.X, pady=5)

ttk.Button(control_strip, text="Refresh Breakdown Matrix", command=show_summary).pack(side=tk.LEFT, padx=5)
ttk.Button(control_strip, text="Render Data Pie Chart Visual", command=show_pie_chart).pack(side=tk.LEFT, padx=5)

summary_text = tk.Text(analytics_container, font=("Consolas", 11), bg=BG_WHITE, fg=TEXT_BLACK, insertbackground="black", bd=0, highlightthickness=1, highlightbackground="#9CA3AF")
summary_text.pack(fill=tk.BOTH, expand=True, pady=10)
summary_text.config(state=tk.DISABLED)


# TAB 4: ADJUSTING LIMIT & FILTER SEARCH
tab4_container = ttk.Frame(tab4, padding=15)
tab4_container.pack(fill=tk.BOTH, expand=True)

limit_box = ttk.LabelFrame(tab4_container, text=" Modify Monthly Budget Ceiling Target ")
limit_box.pack(fill=tk.X, pady=(0, 15))

ttk.Label(limit_box, text="Set New Limit Ceiling (₹):").pack(side=tk.LEFT, padx=10, pady=10)
budget_setup_entry = ttk.Entry(limit_box, width=20)
budget_setup_entry.pack(side=tk.LEFT, padx=10, pady=10)
ttk.Button(limit_box, text="Apply Limit", command=update_budget_setting).pack(side=tk.LEFT, padx=10, pady=10)

search_box = ttk.LabelFrame(tab4_container, text=" Filter Entries via Category Query ")
search_box.pack(fill=tk.BOTH, expand=True)

search_top_row = ttk.Frame(search_box)
search_top_row.pack(fill=tk.X, pady=10, padx=10)

ttk.Label(search_top_row, text="Target Query:").pack(side=tk.LEFT, padx=5)
search_entry = ttk.Entry(search_top_row, width=30)
search_entry.pack(side=tk.LEFT, padx=5)
ttk.Button(search_top_row, text="Search Logs", command=perform_search).pack(side=tk.LEFT, padx=5)

search_tree = ttk.Treeview(search_box, columns=("Date", "Category", "Amount"), show="headings")
search_tree.heading("Date", text="Timestamp")
search_tree.heading("Category", text="Category")
search_tree.heading("Amount", text="Amount (₹)")
search_tree.column("Date", width=180, anchor="center")
search_tree.column("Category", width=260, anchor="w")
search_tree.column("Amount", width=120, anchor="e")
search_tree.pack(fill=tk.BOTH, expand=True, padx=10, pady=10)


def on_tab_switch(event):
    selected_tab_text = event.widget.tab(event.widget.select(), "text").strip()
    if selected_tab_text == "View Log":
        refresh_all_views()
    elif selected_tab_text == "Analytics":
        show_summary()

notebook.bind("<<NotebookTabChanged>>", on_tab_switch)

refresh_all_views()
root.mainloop()