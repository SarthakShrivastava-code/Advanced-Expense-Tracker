# Advanced Expense Tracker (Capital Ledger Engine)

A desktop application built with Python and Tkinter for tracking daily expenses, managing monthly budget ceilings, and visualizing category analytics.

## Features
- **Interactive GUI**: Tabbed interface built using Tkinter and `ttk` styles.
- **Budget Tracking**: Real-time budget health header with dynamic visual warnings when remaining balance is low or exceeded.
- **Analytics & Data Visuals**: Category-wise summary matrix and pie chart distribution rendered via Matplotlib.
- **Data Persistence & Export**: Automatic local storage in CSV format with one-click export to Excel (`.xlsx`) via OpenPyXL.
- **Search & Filter**: Search logged transactions by category keywords.

## Tech Stack
- **Language**: Python 3
- **GUI Framework**: Tkinter / ttk
- **Data Visualization**: Matplotlib
- **Data Handling**: CSV, OpenPyXL

## Setup & Running
1. **Clone the repository:**
   ```bash
   git clone https://github.com/YOUR-USERNAME/Advanced-Expense-Tracker.git
   cd Advanced-Expense-Tracker
   ```

2. **Create and activate a virtual environment (optional but recommended):**
   ```bash
   python -m venv venv
   # On Windows:
   venv\Scripts\activate
   # On macOS/Linux:
   source venv/bin/activate
   ```

3. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

4. **Run the application:**
   ```bash
   python main.py
   ```

## Project Structure
- `main` - Application entry point
- `requirements.txt` - Python dependencies
- `.gitignore` - Ignores local generated data and cache files

## Notes
- Local expense data is stored as CSV files during use.
- Exported reports can be saved in Excel format for sharing or offline analysis.
