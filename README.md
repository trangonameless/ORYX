from tkinter import *
from PIL import ImageTk, Image
from bs4 import BeautifulSoup
import requests
from requests.exceptions import HTTPError
import re
import sqlite3
import matplotlib.pyplot as plt
import time


equipmentall = []
casualties = dict()
data_updated = False
conn = sqlite3.connect("Russia_Cassulties.db")
cursor = conn.cursor()
words = ['Tanks', 'Armoured Fighting Vehicles', 'Aircraft', 'Helicopters']
root = Tk()
root.title('IMAGE')
tanks = 0
ArmouredFightingVehicles = 0
Aircraft = 0
Helicopters = 0

def data_base_actualization(tanks, ArmouredFightingVehicles, Aircraft, Helicopters):
    conn = sqlite3.connect("Russia_Cassulties.db")
    cursor = conn.cursor()

    # Create the table to store the data
    cursor.execute('''
           CREATE TABLE IF NOT EXISTS Russia_Cassulties (
               timestamp TEXT,
               Tanks INT,
               ArmouredFightingVehicles INT,
               Aircraft INT,
               Helicopters INT
           )
       ''')

    # Getting time
    now = time.localtime()
    t = time.strftime("%Y-%m-%d %H:%M:%S")


    # Store the values in the database
    cursor.execute("INSERT INTO Russia_Cassulties (timestamp, Tanks,ArmouredFightingVehicles,Aircraft,Helicopters ) VALUES (?, ?, ? , ?, ?)",
                   (t, tanks, ArmouredFightingVehicles,Aircraft, Helicopters))
    conn.commit()

def get_data_by_datapoint():
    base_url = 'https://www.oryxspioenkop.com/2022/02/attack-on-europe-documenting-equipment.html'
    try:
        response = requests.get(base_url, timeout=3.0)
        response.raise_for_status()
    except HTTPError as http_err:
        print(f'Błąd HTTP: {http_err}')
    except Exception as err:
        print(f'Inny wyjątek: {err}')
    else:
        print('OK!')
        soup = BeautifulSoup(response.content, 'html.parser')
        oryx = soup.get_text()
        return oryx


def data_actualization():
    oryx = get_data_by_datapoint()
    with open('oryx.txt', 'w', encoding='utf-8') as file:
        file.write(oryx)
    global data_updated
    data_updated = True


def extract_text(content, word):
    pattern = fr'{word} \((.*?)\)'
    match = re.search(pattern, content)
    if match:
        extracted_text = match.group(1)
        equipmentall = extracted_text.split()
        equipment = equipmentall[0]
        equipment = equipment[:-1]
        return equipment
    else:
        print("No match found.")
        return None


def extracting():
    with open('oryx.txt', 'r', encoding='utf-8') as file:
        content = file.read()
    for word in words:
        equipmentall.append(extract_text(content, word))

    i = 0

    for word in words:
        casualties[word] = equipmentall[i]
        i += 1
    return casualties


def button_function(number):
    if data_updated == False:
        # Retrieve the data from the database
        cursor.execute(
            "SELECT timestamp, Tanks, ArmouredFightingVehicles, Aircraft, Helicopters FROM Russia_Cassulties ORDER BY timestamp DESC LIMIT 1")
        cass_data = cursor.fetchone()

        if cass_data:
            my_label.configure(text=cass_data[number])
        else:
            my_label.configure(text="No data found in the database.")
    if data_updated == True:
        cursor.execute(
            "SELECT timestamp, Tanks, ArmouredFightingVehicles, Aircraft, Helicopters FROM Russia_Cassulties")
        cass_data = cursor.fetchall()

        if cass_data:
            last_record = cass_data[-1]
            second_last_record = cass_data[-2]
            my_label.configure(text=f'{last_record[number]} + {last_record[number] - second_last_record[number]}')
        else:
            my_label.configure(text="No data found in the database.")


def plot(number):
    cursor.execute(
        "SELECT timestamp, Tanks, ArmouredFightingVehicles, Aircraft, Helicopters FROM Russia_Cassulties")

    cass_data = cursor.fetchall()
    tanks = []
    armoured_vehicles = []
    aircraft = []
    helicopters = []
    time = []

    for x in cass_data:
        if number == 1:
            tanks.append(x[1])
            time.append(x[0])
        elif number == 2:
            armoured_vehicles.append(x[2])
            time.append(x[0])
        elif number == 3:
            aircraft.append(x[3])
            time.append(x[0])
        elif number == 4:
            helicopters.append(x[4])
            time.append(x[0])


    x_indexes = range(len(time))

    # Tworzenie wykresu

    plt.figure(figsize=(12, 6))  # Rozmiar wykresu
    if number == 1:
        plt.plot(x_indexes, tanks, label='Tanks')
    elif number == 2:
        plt.plot(x_indexes, armoured_vehicles, label='Armoured Vehicles')
    elif number == 3:
        plt.plot(x_indexes, aircraft, label='Aircraft')
    elif number == 4:
        plt.plot(x_indexes, helicopters, label='Helicopters')

    # Ustawienie etykiet na osi x
    plt.xticks(x_indexes, time, rotation=45)
    plt.xlabel('Time')

    # Dodanie legendy
    plt.legend()

    # Dodatkowe etykiety i tytuł wykresu
    plt.ylabel('Count')
    plt.title('Military Equipment Over Time')

    plt.tight_layout()
    plt.show()
    time.clear()


my_pic1 = ImageTk.PhotoImage(Image.open("tank.jpg"))
my_pic2 = ImageTk.PhotoImage(Image.open("d_tank.jpg"))
my_pic3 = ImageTk.PhotoImage(Image.open("mig.jpg"))
my_pic4 = ImageTk.PhotoImage(Image.open("d_mig.png"))
my_pic5 = ImageTk.PhotoImage(Image.open("bwp.jpg"))
my_pic6 = ImageTk.PhotoImage(Image.open("d_bwp.jpg"))
my_pic7 = ImageTk.PhotoImage(Image.open("heli.jpg"))
my_pic8 = ImageTk.PhotoImage(Image.open("d_heli.jpg"))


image_list = [my_pic1, my_pic2,my_pic3,my_pic4,my_pic5,my_pic6,my_pic7,my_pic8]

my_label = Label(text="", fg="red",font=("Arial", 20))
my_label.grid(row=0,column=2, columnspan=3)


button1 = Button(root, text='Download actual russian army casualties from oryx')
button1.grid(row=2, column=0, columnspan=2)
button2 = Button(root, image=my_pic1)
button2.grid(row=4, column=0, columnspan=2)
button3 = Button(root, image=my_pic3)
button3.grid(row=6, column=0, columnspan=2)
button4 = Button(root, image=my_pic5)
button4.grid(row=4, column=2, columnspan=2)
button5 = Button(root, image=my_pic7)
button5.grid(row=6, column=2, columnspan=2)
button6 = Button(root, text="Russian Tank losses over time")
button6.grid(row=3, column=0, columnspan=2)
button7 = Button(root, text="Russian Armoured Vehicles losses over time")
button7.grid(row=2, column=3, columnspan=2)
button8 = Button(root, text="Russian Aircraft losses over time")
button8.grid(row=2, column=8, columnspan=2)
button9 = Button(root, text="Russian Helicopters losses over time")
button9.grid(row=3, column=8, columnspan=2)

def button_1():
    data_actualization()
    casualties = extracting()
    values_list = list(casualties.values())
    tanks = int(values_list[0])
    ArmouredFightingVehicles = int(values_list[1])
    Aircraft = int(values_list[2])
    Helicopters = int(values_list[3])
    data_base_actualization(tanks, ArmouredFightingVehicles, Aircraft, Helicopters)
    data_updated = True
    return tanks, ArmouredFightingVehicles, Aircraft, Helicopters, data_updated
def button_2(image_number):
    button2.configure(image=image_list[image_number])
    button_function(1)
def button_3(image_number):
    button3.configure(image=image_list[image_number])
    button_function(3)
def button_4(image_number):
    button4.configure(image=image_list[image_number])
    button_function(2)
def button_5(image_number):
    button5.configure(image=image_list[image_number])
    button_function(4)
def button_6():
    plot(1)
def button_7():
    plot(2)
def button_8():
    plot(3)
def button_9():
    plot(4)

button1.config(command=lambda : button_1())
button2.config(command=lambda: button_2(1))
button3.config(command=lambda: button_3(3))
button4.config(command=lambda: button_4(5))
button5.config(command=lambda: button_5(7))
button6.config(command=lambda: button_6())
button7.config(command=lambda: button_7())
button8.config(command=lambda: button_8())
button9.config(command=lambda: button_9())




root.mainloop()
