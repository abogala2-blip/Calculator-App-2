# Calculator-App-2

import tkinter as tk
import math

# ---------------- Color & Font Settings ---------------- #
BG_COLOR = "#222831"
BTN_COLOR = "#393E46"
BTN_HOVER = "#00ADB5"
ACCENT_COLOR = "#00FFF5"
DISPLAY_COLOR = "#EEEEEE"
FONT = ("Segoe UI", 18)
BTN_FONT = ("Segoe UI", 16, "bold")

# ---------------- Safe Evaluation ---------------- #
allowed_names = {k: v for k, v in math.__dict__.items()}

def safe_eval(expr):
    try:
        return eval(expr, {"__builtins__": {}}, allowed_names)
    except Exception:
        return "Error"

# ---------------- Calculator Class ---------------- #
class Calculator(tk.Tk):
    def __init__(self):
        super().__init__()
        self.title("Modern Calculator")
        self.geometry("360x530")
        self.configure(bg=BG_COLOR)
        self.resizable(False, False)
        self.expression = ""
        self.create_widgets()
        self.bind("<Key>", self.on_key)

    def create_widgets(self):
        self.display_var = tk.StringVar()
        self.display = tk.Entry(
            self,
            textvariable=self.display_var,
            font=FONT,
            bg=DISPLAY_COLOR,
            fg=BG_COLOR,
            bd=0,
            relief=tk.FLAT,
            justify='right'
        )
        self.display.place(x=20, y=30, width=320, height=60)
        self.display.config(state="disabled")

        btn_texts = [
            ("C", "⌫", "%", "/"),
            ("7", "8", "9", "*"),
            ("4", "5", "6", "-"),
            ("1", "2", "3", "+"),
            ("±", "0", ".", "=")
        ]
        for i, row in enumerate(btn_texts):
            for j, txt in enumerate(row):
                btn = tk.Button(
                    self,
                    text=txt,
                    font=BTN_FONT,
                    bg=BTN_COLOR,
                    fg=ACCENT_COLOR if txt in "/*-+=." else DISPLAY_COLOR,
                    bd=0,
                    activebackground=BTN_HOVER,
                    activeforeground=DISPLAY_COLOR,
                    command=lambda t=txt: self.on_button_click(t)
                )
                btn.place(x=20 + j*80, y=110 + i*80, width=70, height=70)
                btn.bind("<Enter>", lambda e, b=btn: b.config(bg=BTN_HOVER))
                btn.bind("<Leave>", lambda e, b=btn: b.config(bg=BTN_COLOR))

    def on_button_click(self, char):
        if char == "C":
            self.expression = ""
        elif char == "⌫":
            self.expression = self.expression[:-1]
        elif char == "=":
            expr = self.expression.replace("%", "/100")
            result = safe_eval(expr)
            if result != "Error":
                result = str(result)
                if result.endswith(".0"):
                    result = result[:-2]
            self.expression = result
        elif char == "±":
            if self.expression.startswith("-"):
                self.expression = self.expression[1:]
            else:
                self.expression = "-" + self.expression
        else:
            if char in "+-*/." and self.expression and self.expression[-1] in "+-*/.":
                return
            self.expression += char
        self.update_display()

    def update_display(self):
        self.display.config(state="normal")
        self.display_var.set(self.expression)
        self.display.config(state="disabled")

    def on_key(self, event):
        if event.char.isdigit() or event.char in "+-*/.%":
            self.on_button_click(event.char)
        elif event.keysym == "Return":
            self.on_button_click("=")
        elif event.keysym == "BackSpace":
            self.on_button_click("⌫")

if __name__ == "__main__":
    Calculator().mainloop()
