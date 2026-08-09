import sqlite3
import time
import pyperclip
from datetime import datetime

DB_NAME = "clipboard.db"


class ClipboardManager:
    def __init__(self):
        self.conn = sqlite3.connect(DdB_NAME)
        self.create_table()

    def create_table(self):
        self.conn.execute("""
        CREATE TABLE IF NOT EXISTS history (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            content TEXT UNIQUE,
            created_at TEXT
        )
        """)
        self.conn.commit()

    def save(self, text):
        try:
            self.conn.execute(
                "INSERT INTO history(content, created_at) VALUES(?, ?)",
                (text, datetime.now().isoformat())
            )
            self.conn.commit()
            print("Saved")
        except sqlite3.IntegrityError:
            pass

    def search(self, keyword):
        cursor = self.conn.execute(
            "SELECT content, created_at FROM history WHERE content LIKE ? ORDER BY id DESC",
            (f"%{keyword}%",)
        )

        results = cursor.fetchall()

        if not results:
            print("No results")
            return

        print("\nResults:\n")

        for content, date in results:
            print(f"[{date}]")
            print(content[:150])
            print("-" * 50)

    def show_latest(self, limit=10):
        cursor = self.conn.execute(
            "SELECT content, created_at FROM history ORDER BY id DESC LIMIT ?",
            (limit,)
        )

        for content, date in cursor.fetchall():
            print(f"[{date}]")
            print(content[:100])
            print("-" * 50)


def monitor():
    manager = ClipboardManager()

    last_text = ""

    print("Monitoring clipboard...")

    while True:
        try:
            current = pyperclip.paste()

            if current and current != last_text:
                manager.save(current)
                last_text = current

            time.sleep(1)

        except KeyboardInterrupt:
            print("\nStopped")
            break


def menu():
    manager = ClipboardManager()

    while True:
        print("\n1. Monitor Clipboard")
        print("2. Search History")
        print("3. Show Latest")
        print("4. Exit")

        choice = input("\nSelect: ")

        if choice == "1":
            monitor()

        elif choice == "2":
            keyword = input("Keyword: ")
            manager.search(keyword)

        elif choice == "3":
            manager.show_latest()

        elif choice == "4":
            break


if __name__ == "__main__":
    menu()
