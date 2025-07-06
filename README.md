import tkinter as tk
from tkinter import messagebox
import requests

# أدخل مفتاح API الخاص بك هنا
API_KEY = "ضع_مفتاحك_هنا"

# دالة لجلب المباريات
def get_matches():
    url = "https://v3.football.api-sports.io/fixtures?date=2025-07-06"
    headers = {
        "x-apisports-key": API_KEY
    }
    response = requests.get(url, headers=headers)
    if response.status_code != 200:
        messagebox.showerror("خطأ", "فشل الاتصال بالسيرفر.")
        return []
    data = response.json()
    matches = []
    for item in data["response"]:
        team1 = item["teams"]["home"]["name"]
        team2 = item["teams"]["away"]["name"]
        score = item["goals"]["home"], item["goals"]["away"]
        matches.append({
            "team1": team1,
            "team2": team2,
            "score": f"{score[0]} - {score[1]}"
        })
    return matches

# دالة لعرض النتيجة
def show_result(match):
    result = f"{match['team1']} vs {match['team2']}\nالنتيجة: {match['score']}"
    messagebox.showinfo("نتيجة المباراة", result)

# واجهة التطبيق
root = tk.Tk()
root.title("نتائج المباريات اليوم")
root.geometry("350x500")

title = tk.Label(root, text="مباريات اليوم", font=("Arial", 16, "bold"))
title.pack(pady=10)

# زر تحميل المباريات
def load_matches():
    for widget in frame.winfo_children():
        widget.destroy()
    matches = get_matches()
    if not matches:
        label = tk.Label(frame, text="لا توجد مباريات اليوم.", font=("Arial", 12))
        label.pack(pady=20)
        return
    for match in matches:
        btn = tk.Button(frame,
                        text=f"{match['team1']} vs {match['team2']}",
                        font=("Arial", 12),
                        command=lambda m=match: show_result(m))
        btn.pack(pady=5)

frame = tk.Frame(root)
frame.pack()

load_btn = tk.Button(root, text="تحديث النتائج", font=("Arial", 12), command=load_matches)
load_btn.pack(pady=10)

# تحميل النتائج مباشرة عند بدء التشغيل
load_matches()

root.mainloop()
