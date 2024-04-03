# Automated-Sorting-Machine
Automated Sorting Machine Code
import cv2
import serial
from ultralytics import YOLO
from circle_detection import detect_circle_get_radius


def get_radius_quality(model, n_samples):
    cap = cv2.VideoCapture(0)
    count_samples = 0
    sum_radius = 0.0
    not_rotten_confidence = 0.0
    rotten_confidence = 0.0
    final_radius = 0.0
    while cap.isOpened():
        # Read a frame from the video
        ret, frame = cap.read()

        if not ret:
            continue

        # Process the frame
        processed_frame, radius = detect_circle_get_radius(frame)
        results = model(frame, show = True, conf = 0.25, max_det = 1)
        # Display the resulting frame
        # cv2.imshow('Tomato Detection', processed_frame)
        count_samples += 1
        sum_radius += radius
        # print(results)
        if (len(results[0].boxes) > 0 and len(results[0].boxes[0]) > 0):
            not_rotten_confidence += float(results[0].boxes.conf[0])
        else :
            not_rotten_confidence += float(0)

        if (len(results[0].boxes) > 0 and len(results[0].boxes[1]) > 0):
            rotten_confidence += float(results[0].boxes.conf[1])
        else :
            rotten_confidence += float(0)

        if count_samples == n_samples:
            final_radius = sum_radius / n_samples
            not_rotten_confidence = not_rotten_confidence / n_samples
            rotten_confidence = rotten_confidence / n_samples
            break
        # Break the loop when 'q' key is pressed
        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

    cap.release()
    cv2.destroyAllWindows()
    return final_radius, not_rotten_confidence, rotten_confidence
    # Release the video capture object and close the OpenCV window


def main():
    model = YOLO("YOLO/best2.pt")
    print("YOLO model loaded")
    # ser = serial.Serial(
    #     'COM3', 9600
    # )
    print("Serially connected")
    while True:

        # s = ser.readline().decode("utf-8")
        # s = s[:s.find('\r')]
        # print("Input from arduino ", s)

        radius, not_rotten_conf, rotten_conf = get_radius_quality(model, n_samples=5)
        print(f"Radius of tomato: {radius:0.2f} px")
        # print(f"Not Rotten tomatoes confidence: {confidence:0.2f} ")
        if (max(not_rotten_conf,rotten_conf) <= 0.25 or radius <= 10.0):
            print("No tomato detected")
        elif (rotten_conf > not_rotten_conf and radius > 10.0):
            print("Rotten tomatoes detected")
            # ser.write("-1".encode("utf-8"))
        elif (rotten_conf <= not_rotten_conf and radius > 10.0):
            if radius > 120 and radius < 135 :
                print("Best tomatoes detected")
                # ser.write("1".encode("utf-8"))
            else:
                print("Good tomatoes detected")
                # ser.write("2".encode("utf-8"))


if _name_ == '_main_':
    main()
