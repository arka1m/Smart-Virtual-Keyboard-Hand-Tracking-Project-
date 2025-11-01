# Smart-Virtual-Keyboard-Hand-Tracking-Project-

 A real-time Virtual Keyboard built using OpenCV, Mediapipe, and cvzone — control your keyboard with hand gestures instead of keys!



Code :

import os
os.environ['TF_CPP_MIN_LOG_LEVEL'] = '3'

import cv2
from cvzone.HandTrackingModule import HandDetector

cap = cv2.VideoCapture(0, cv2.CAP_DSHOW)
cap.set(3, 1280)
cap.set(4, 800)

detector = HandDetector(detectionCon=0.8, maxHands=1)

finalText = ""
delayCounter = 0
lastKey = None

keyboardRows = [
    ["Q", "W", "E", "R", "T", "Y", "U", "I", "O", "P"],
    ["A", "S", "D", "F", "G", "H", "J", "K", "L", ";", "'"],
    ["Z", "X", "C", "V", "B", "N", "M", ",", ".", "/", "<"],
    ["Space", "Backspace"]
]

button_w, button_h = 60, 60
button_gap = 10
start_y = 280

text_color = (255, 255, 255)
overlay_color = (0, 0, 0)
overlay_alpha = 0.5

row_colors = [
    ((0, 120, 255), (0, 255, 120)),
    ((255, 0, 255), (255, 128, 192)),
    ((255, 255, 0), (0, 255, 255))
]
spacebar_color = (255, 180, 100)
backspace_color = (50, 50, 50)
hover_color = (0, 255, 255)

def interpolate_color(c1, c2, f):
    return (int(c1[0] + (c2[0]-c1[0])*f),
            int(c1[1] + (c2[1]-c1[1])*f),
            int(c1[2] + (c2[2]-c1[2])*f))

def draw_keyboard(img):
    overlay = img.copy()
    h, w = img.shape[:2]
    cv2.rectangle(overlay, (0, start_y-30), (w, h), overlay_color, -1)
    cv2.addWeighted(overlay, overlay_alpha, img, 1 - overlay_alpha, 0, img)
    for rowIndex, row in enumerate(keyboardRows):
        if row == ["Space", "Backspace"]:
            width_space = button_w*4 + button_gap*3
            width_backspace = button_w*2 + button_gap
            total_width = width_space + button_gap + width_backspace
            start_x = (w - total_width)//2
            y = start_y + rowIndex*(button_h + button_gap)
            cv2.rectangle(img, (start_x, y), (start_x+width_space, y+button_h), spacebar_color, -1)
            cv2.putText(img, "Space", (start_x+30, y+45), cv2.FONT_HERSHEY_PLAIN, 3, text_color, 3)
            x_back = start_x + width_space + button_gap
            cv2.rectangle(img, (x_back, y), (x_back+width_backspace, y+button_h), backspace_color, -1)
            cv2.putText(img, "BS", (x_back+15, y+45), cv2.FONT_HERSHEY_PLAIN, 3, text_color, 3)
        else:
            total_row_width = len(row)*button_w + (len(row)-1)*button_gap
            start_x = (w - total_row_width)//2
            start_color, end_color = row_colors[rowIndex]
            for keyIndex, key in enumerate(row):
                factor = keyIndex / (len(row)-1) if len(row) > 1 else 0
                color = interpolate_color(start_color, end_color, factor)
                x = start_x + keyIndex*(button_w + button_gap)
                y = start_y + rowIndex*(button_h + button_gap)
                cv2.rectangle(img, (x, y), (x+button_w, y+button_h), color, -1)
                ts = cv2.getTextSize(key, cv2.FONT_HERSHEY_PLAIN, 2, 2)[0]
                cv2.putText(img, key, (x + (button_w-ts[0])//2, y + (button_h+ts[1])//2), cv2.FONT_HERSHEY_PLAIN, 2, text_color, 2)

def highlight_key(img, key, rowIndex, keyIndex):
    h, w = img.shape[:2]
    if key in ["Space", "Backspace"]:
        width_space = button_w*4 + button_gap*3
        width_backspace = button_w*2 + button_gap
        total_width = width_space + button_gap + width_backspace
        start_x = (w - total_width)//2
        y = start_y + rowIndex*(button_h + button_gap)
        if key == "Space":
            cv2.rectangle(img, (start_x, y), (start_x+width_space, y+button_h), hover_color, -1)
            cv2.putText(img, "Space", (start_x+30, y+45), cv2.FONT_HERSHEY_PLAIN, 3, text_color, 3)
        else:
            x_back = start_x + width_space + button_gap
            cv2.rectangle(img, (x_back, y), (x_back+width_backspace, y+button_h), hover_color, -1)
            cv2.putText(img, "BS", (x_back+15, y+45), cv2.FONT_HERSHEY_PLAIN, 3, text_color, 3)
    else:
        total_row_width = len(keyboardRows[rowIndex])*button_w + (len(keyboardRows[rowIndex])-1)*button_gap
        start_x = (w - total_row_width)//2
        x = start_x + keyIndex*(button_w + button_gap)
        y = start_y + rowIndex*(button_h + button_gap)
        cv2.rectangle(img, (x, y), (x+button_w, y+button_h), hover_color, -1)
        ts = cv2.getTextSize(key, cv2.FONT_HERSHEY_PLAIN, 2, 2)[0]
        cv2.putText(img, key, (x + (button_w-ts[0])//2, y + (button_h+ts[1])//2), cv2.FONT_HERSHEY_PLAIN, 2, text_color, 2)

while True:
    success, img = cap.read()
    if not success:
        break
    img = cv2.flip(img, 1)
    hands, img = detector.findHands(img)
    draw_keyboard(img)
    if hands:
        lmList = hands[0]["lmList"]
        if len(lmList) > 12:
            x1, y1 = lmList[8][:2]
            x2, y2 = lmList[12][:2]
            distance = ((x2-x1)*2 + (y2-y1)2)*0.5
            for rowIndex, row in enumerate(keyboardRows):
                if row == ["Space", "Backspace"]:
                    width_space = button_w*4 + button_gap*3
                    width_backspace = button_w*2 + button_gap
                    total_width = width_space + button_gap + width_backspace
                    start_x = (img.shape[1]-total_width)//2
                    y = start_y + rowIndex*(button_h + button_gap)
                    if start_x < x1 < start_x+width_space and y < y1 < y+button_h:
                        highlight_key(img, "Space", rowIndex, 0)
                        if distance < 40 and (delayCounter == 0 or lastKey != "Space"):
                            finalText += " "
                            lastKey = "Space"
                            delayCounter = 1
                    x_back = start_x + width_space + button_gap
                    if x_back < x1 < x_back+width_backspace and y < y1 < y+button_h:
                        highlight_key(img, "Backspace", rowIndex, 1)
                        if distance < 40 and (delayCounter == 0 or lastKey != "Backspace"):
                            finalText = finalText[:-1]
                            lastKey = "Backspace"
                            delayCounter = 1
                else:
                    total_row_width = len(row)*button_w + (len(row)-1)*button_gap
                    start_x = (img.shape[1]-total_row_width)//2
                    for keyIndex, key in enumerate(row):
                        x = start_x + keyIndex*(button_w + button_gap)
                        y = start_y + rowIndex*(button_h + button_gap)
                        if x < x1 < x+button_w and y < y1 < y+button_h:
                            highlight_key(img, key, rowIndex, keyIndex)
                            if distance < 40 and (delayCounter == 0 or lastKey != key):
                                finalText += key
                                lastKey = key
                                delayCounter = 1
    cv2.rectangle(img, (50, 50), (img.shape[1]-50, 150), (50, 50, 50), -1)
    cv2.putText(img, finalText, (60, 120), cv2.FONT_HERSHEY_PLAIN, 3, (255, 255, 255), 3)

    if delayCounter != 0:
        delayCounter += 1
        if delayCounter > 15:
            delayCounter = 0

    cv2.imshow("Virtual Keyboard", img)
    if cv2.waitKey(1) & 0xFF == 27:  # ESC key
        break

cap.release()
cv2.destroyAllWindows()
cv2.waitKey(1)  # Ensures window actually closes




 
 This project turns your webcam into a smart virtual keyboard using AI-powered hand tracking.
With just your fingers, you can hover, select, and type on a digital keyboard displayed on your screen.
It’s a fun and futuristic project combining computer vision, gesture recognition, and Python programming.
 💡 Key Highlights

Real-time hand gesture detection using Mediapipe

Smooth and colorful on-screen keyboard interface

Detects pinch gestures to simulate key presses

Supports typing, space, and backspace functions

Ideal for AI, computer vision, and HCI (Human-Computer Interaction) projects
🧰 Tech Stack

Language: Python

Libraries: OpenCV, Mediapipe, cvzone, NumPy

Hardware: Any standard webcam

💻 Usage

Run the script and allow webcam access.

Use your index finger to point to any key.

Pinch with index + middle finger to “press” the key.

The pressed keys appear in the text area on top of the screen.

🧠 Inspiration

This project explores the concept of contactless typing — a futuristic and hygienic input method that blends AI and Computer Vision.

📚 Ideal For

Students learning OpenCV or Mediapipe

Gesture recognition enthusiasts

Final year or mini-projects in AI / CV / HCI

✨ Sample One-Liners (for GitHub “About” section)

"Virtual keyboard controlled by hand gestures using Mediapipe and OpenCV 👋"

"AI-powered gesture-based typing system 🔠"

"Contactless typing interface using hand tracking 💻🖐️"

"A futuristic on-screen keyboard controlled by finger movements"

"Gesture-controlled virtual keyboard built with Python, OpenCV, and Mediapipe"
