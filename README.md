Problem statement: Workout Tracking App
We input the workouts done by the user and based on that data we calculate the calories spent

import requests
from datetime import datetime
import webbrowser
from tkinter import *


APP_ID = "5c663869"
API_KEY = "9a68c02a386d3641f296ee1b2d073aed"
SHEETY_URL = "https://api.sheety.co/39de4e2da8695938464d35c3b6cfe976/workouts/workouts"
SHEET_HEADERS={"Authorization":"Bearer afihowfoohgo"}


def track_workout(inp, weight, height, age):
    try:

        url = "https://trackapi.nutritionix.com/v2/natural/exercise"
        parameters = {
            "query": inp,
            "weight_kg": weight,
            "height_cm": height,
            "age": age
        }
        headers = {"x-app-id": APP_ID, "x-app-key": API_KEY}

       
        response = requests.post(url=url, json=parameters, headers=headers)
        response.raise_for_status()  
        data = response.json()
        exercises = data["exercises"]


        log_to_sheet(exercises)

    except Exception as e:
        print(f"Error: {e}")






def log_to_sheet(exercises):
    try:
        
        now = datetime.now()



        for exercise in exercises:
            sheet_params = {
                "workout": {
                    "date": now.strftime("%d/%m/%Y"),
                    "time": now.strftime("%H:%M:%S"),
                    "exercise": exercise["name"],
                    "duration": exercise["duration_min"],
                    "calories": exercise["nf_calories"]
                }
            }
            response = requests.post(url=SHEETY_URL, json=sheet_params)
            response.raise_for_status()


        google_sheet_url = "https://docs.google.com/spreadsheets/d/1mZZUtnBNew1gG6E91HwoJjJtXPor6yr9W4-ezxA6-O8"
        print("Opening Google Sheet...")
        webbrowser.open(google_sheet_url)

    except Exception as e:
        print(f"Error while logging to sheet: {e}")



def submit():
    inp = entry_exercise.get()
    weight = float(entry_weight.get())
    height = float(entry_height.get())
    age = int(entry_age.get())

    
    entry_exercise.delete(0, END)

    
    track_workout(inp, weight, height, age)


window = Tk()
window.title('Workout Tracker')
window.config(padx=50, pady=50, bg="yellow")


Label(window, text="Exercise Details:", bg="yellow", pady=5).grid(column=1, row=0)
entry_exercise = Entry(window, width=35)
entry_exercise.grid(column=1, row=1)

Label(window, text="Weight (kg):", bg="yellow", pady=5).grid(column=1, row=2)
entry_weight = Entry(window, width=35)
entry_weight.grid(column=1, row=3)

Label(window, text="Height (cm):", bg="yellow", pady=5).grid(column=1, row=4)
entry_height = Entry(window, width=35)
entry_height.grid(column=1, row=5)

Label(window, text="Age:", bg="yellow", pady=5).grid(column=1, row=6)
entry_age = Entry(window, width=35)
entry_age.grid(column=1, row=7)


button = Button(window, text="Submit", command=submit, bg="blue", fg="white", padx=10, pady=5)
button.grid(column=1, row=8)


window.mainloop()
