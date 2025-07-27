import cv2
from cvzone.HandTrackingModule import HandDetector

class Button:
    def __init__(self, pos, width, height, value):
        self.pos = pos
        self.width = width
        self.height = height
        self.value = value

    def draw(self, img, transparency=0.5):
        overlay = img.copy()
        # Draw button background with transparency
        cv2.rectangle(overlay, self.pos, (self.pos[0] + self.width, self.pos[1] + self.height),
                      (225, 225, 225), cv2.FILLED)
        cv2.addWeighted(overlay, transparency, img, 1 - transparency, 0, img)
        # Draw button outline
        cv2.rectangle(img, self.pos, (self.pos[0] + self.width, self.pos[1] + self.height),
                      (50, 50, 50), 3)
        # Draw button text
        cv2.putText(img, self.value, (self.pos[0] + 30, self.pos[1] + 70), cv2.FONT_HERSHEY_PLAIN,
                    2, (50, 50, 50), 2)

    def checkClick(self, x, y):
        if self.pos[0] < x < self.pos[0] + self.width and self.pos[1] < y < self.pos[1] + self.height:
            # Highlight button when clicked
            cv2.rectangle(img, (self.pos[0] + 3, self.pos[1] + 3),
                          (self.pos[0] + self.width - 3, self.pos[1] + self.height - 3),
                          (255, 255, 255), cv2.FILLED)
            cv2.putText(img, self.value, (self.pos[0] + 25, self.pos[1] + 80), cv2.FONT_HERSHEY_PLAIN,
                        5, (0, 0, 0), 5)
            return True
        return False

# Button Layout
buttonListValues = [['AC', '', '%', '/'],
                    ['7', '8', '9', '*'],
                    ['4', '5', '6', '-'],
                    ['1', '2', '3', '+'],
                    ['+/-', '0', '.', '=']]

# Create Button Objects
buttonList = []
for x in range(4):
    for y in range(5):  # Updated to accommodate 5 rows
        xpos = x * 100 + 50  # Adjusting button positions
        ypos = y * 100 + 150  # Adjusting button positions
        buttonList.append(Button((xpos, ypos), 100, 100, buttonListValues[y][x]))

# Equation variables
myEquation = ''
delayCounter = 0

# Webcam Setup
cap = cv2.VideoCapture(0)
cap.set(3, 1280)  # Width
cap.set(4, 720)   # Height

detector = HandDetector(detectionCon=0.8, maxHands=1)

while True:
    # Capture frame from webcam
    success, img = cap.read()
    if not success:
        print("Failed to capture image from webcam")
        break

    # Flip the image horizontally
    img = cv2.flip(img, 1)

    # Draw the display screen with transparency
    overlay = img.copy()
    cv2.rectangle(overlay, (50, 70), (50 + 400, 70 + 100), (225, 225, 225), cv2.FILLED)
    cv2.addWeighted(overlay, 0.5, img, 0.5, 0, img)  # Make the display screen transparent
    cv2.rectangle(img, (50, 70), (50 + 400, 70 + 100), (50, 50, 50), 3)

    # Draw all the buttons with transparency
    for button in buttonList:
        button.draw(img, transparency=0.5)

    # Detect hands and positions
    hands, img = detector.findHands(img)
    
    if hands:
        # Find the landmarks of the hand
        lmList = hands[0]['lmList']
        if len(lmList) >= 12:  # Ensure there are enough landmarks
            length, _, img = detector.findDistance(lmList[8], lmList[12], img)  # Distance between index and middle finger
            x, y,_ = lmList[8]  # Get the position of the index finger

            # If clicked check which button and perform action
            if length < 50 and delayCounter == 0:  # Click detected
                for i, button in enumerate(buttonList):
                    if button.checkClick(x, y):
                        myValue = buttonListValues[int(i // 4)][i % 4]  # get correct value based on position
                        if myValue == '=':
                            try:
                                myEquation = str(eval(myEquation))
                            except Exception as e:
                                myEquation = 'Error'
                        elif myValue == 'AC':
                            myEquation = ''
                        elif myValue == '':
                            myEquation = myEquation[:-1]  # Backspace functionality
                        elif myValue == '%':
                            try:
                                if myEquation:
                                    myEquation = str(eval(myEquation) / 100)  # Calculate percentage
                            except Exception:
                                myEquation = 'Error'
                        elif myValue == '+/-':
                            if myEquation and myEquation[-1] not in '+-*/':
                                try:
                                    if myEquation[0] == '-':
                                        myEquation = myEquation[1:]  # Remove the negative sign
                                    else:
                                        myEquation = '-' + myEquation  # Add a negative sign
                                except Exception:
                                    myEquation = 'Error'
                        else:
                            myEquation += myValue
                        delayCounter = 1  # Prevent multiple clicks

    # to avoid multiple clicks
    if delayCounter != 0:
        delayCounter += 1
        if delayCounter > 10:
            delayCounter = 0

    # Write the Final answer
    cv2.putText(img, myEquation, (60, 130), cv2.FONT_HERSHEY_PLAIN,
                3, (0, 0, 0), 3)

    # Display the image
    cv2.imshow("Image", img)
    
    key = cv2.waitKey(1)
    if key == ord('q'):  # Press 'q' to exit
        break

# Release the webcam and close all OpenCV windows
cap.release()
cv2.destroyAllWindows()
# somith-kumar
virtual calculator by using hand detecting
