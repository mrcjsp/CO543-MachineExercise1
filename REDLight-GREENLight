import cv2
import numpy as np
import random

# --------------------------
# Configuration Parameters
# --------------------------
FRAME_WIDTH = 640
BLUR_KERNEL = (5, 5)
GREEN_MOVE_THRESHOLD = 0.04 
RED_MOVE_THRESHOLD = 0.055 

# Randomized durations (ms)
def random_duration(min_ms, max_ms):
    return random.randint(min_ms, max_ms)

# Timer variables (convert to seconds for OpenCV)
green_duration = random_duration(2600, 4200) / 1000.0
red_duration = random_duration(1700, 2900) / 1000.0
red_grace_ms = 650 / 1000.0
idle_warning_ms = 1800 / 1000.0
idle_death_ms = 3600 / 1000.0

# --------------------------
# State Machine Setup
# --------------------------
STATES = ["GREEN", "RED", "WARNING", "DEAD"]
current_state = "GREEN"

# Timers
state_timer = 0.0
idle_timer = 0.0
red_grace_timer = 0.0
prev_gray = None

# --------------------------
# Main Webcam Processing
# --------------------------
cap = cv2.VideoCapture(0)
if not cap.isOpened():
    raise ValueError("Could not open webcam")

# Set frame width (height adjusts proportionally)
cap.set(cv2.CAP_PROP_FRAME_WIDTH, FRAME_WIDTH)

fps = cap.get(cv2.CAP_PROP_FPS) or 30.0  
frame_delay = 1.0 / fps

while True:
    ret, frame = cap.read()
    if not ret:
        break

    # ----------------------
    # A. Input and Preprocessing
    # ----------------------
    
    height = int(frame.shape[0] * (FRAME_WIDTH / frame.shape[1]))
    resized = cv2.resize(frame, (FRAME_WIDTH, height))

    gray = cv2.cvtColor(resized, cv2.COLOR_BGR2GRAY)

    gray = cv2.GaussianBlur(gray, BLUR_KERNEL, 0)

    # ----------------------
    # B. Motion Score Calculation
    # ----------------------
    motion_score = 0.0
    if prev_gray is not None:
        diff = cv2.absdiff(gray, prev_gray)
        motion_score = np.mean(diff) / 255.0
    prev_gray = gray.copy()

    # ----------------------
    # C. RLGL State Machine
    # ----------------------
   
    state_timer += frame_delay
    if current_state == "GREEN" or current_state == "WARNING":
        idle_timer += frame_delay if motion_score < GREEN_MOVE_THRESHOLD else 0.0

    # State logic - D. Rules
    if current_state == "GREEN":
    
        if idle_timer >= idle_death_ms:
            current_state = "DEAD"
        elif idle_timer >= idle_warning_ms:
            current_state = "WARNING"
       
        elif state_timer >= green_duration:
            current_state = "RED"
            state_timer = 0.0
            red_grace_timer = 0.0
            idle_timer = 0.0 

    elif current_state == "RED":
        red_grace_timer += frame_delay
        
        if red_grace_timer >= red_grace_ms:
            if motion_score > RED_MOVE_THRESHOLD:
                current_state = "DEAD"
       
        if state_timer >= red_duration:
            print("SUCCESS: Red state completed without violation!")
            break  

    elif current_state == "WARNING":
       
        if idle_timer >= idle_death_ms:
            current_state = "DEAD"
      
        elif motion_score >= GREEN_MOVE_THRESHOLD:
            current_state = "GREEN"
            idle_timer = 0.0

    elif current_state == "DEAD":
        print("FAILURE: State entered DEAD!")
        break

    # ----------------------
    # Visualization (Optional)
    # ----------------------
    cv2.putText(resized, f"State: {current_state}", (10, 30),
                cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 255, 0) if current_state == "GREEN" else (0, 0, 255) if current_state == "RED" else (0, 255, 255), 2)
    cv2.putText(resized, f"Motion Score: {motion_score:.4f}", (10, 70),
                cv2.FONT_HERSHEY_SIMPLEX, 0.7, (255, 255, 255), 2)
    cv2.imshow("RLGL State Machine", resized)

    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

# Cleanup
cap.release()
cv2.destroyAllWindows()
