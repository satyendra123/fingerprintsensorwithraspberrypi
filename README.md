python
Copy code
import time
from pyfingerprint.pyfingerprint import PyFingerprint
import RPi.GPIO as gpio

enrol = 5
delet = 6
inc = 13
dec = 19
led = 26

gpio.setwarnings(False)
gpio.setmode(gpio.BCM)

gpio.setup(enrol, gpio.IN, pull_up_down=gpio.PUD_UP)
gpio.setup(delet, gpio.IN, pull_up_down=gpio.PUD_UP)
gpio.setup(inc, gpio.IN, pull_up_down=gpio.PUD_UP)
gpio.setup(dec, gpio.IN, pull_up_down=gpio.PUD_UP)
gpio.setup(led, gpio.OUT)

try:
    f = PyFingerprint('/dev/ttyUSB0', 57600, 0xFFFFFFFF, 0x00000000)

    if not f.verifyPassword():
        raise ValueError('The given fingerprint sensor password is wrong!')

except Exception as e:
    print('Exception message: ' + str(e))
    exit(1)

def enrollFinger():
    print('Enrolling Finger')
    time.sleep(2)
    print('Waiting for finger...')
    print('Place Finger')
    while not f.readImage():
        pass
    f.convertImage(0x01)
    result = f.searchTemplate()
    positionNumber = result[0]
    if positionNumber >= 0:
        print('Template already exists at position #' + str(positionNumber))
        print('Finger Already Exists')
        time.sleep(2)
        return
    print('Remove finger...')
    print('Waiting for same finger again...')
    print('Place Finger Again')
    while not f.readImage():
        pass
    f.convertImage(0x02)
    if f.compareCharacteristics() != 0:
        print('Fingers do not match')
        print('Finger Did not Mactched')
        time.sleep(2)
        return
    f.createTemplate()
    positionNumber = f.storeTemplate()
    print('Finger enrolled successfully!')
    print('Stored at Pos:' + str(positionNumber))
    print('New template position #' + str(positionNumber))
    time.sleep(2)

def searchFinger():
    try:
        print('Waiting for finger...')
        while not f.readImage():
            time.sleep(.5)
        f.convertImage(0x01)
        result = f.searchTemplate()
        positionNumber = result[0]
        accuracyScore = result[1]
        if positionNumber == -1 :
            print('No match found!')
            print('No Match Found')
            time.sleep(2)
            return
        else:
            print('Found template at position #' + str(positionNumber))
            print('Found at Pos:' + str(positionNumber))
            time.sleep(2)

    except Exception as e:
        print('Operation failed!')
        print('Exception message: ' + str(e))
        exit(1)

def deleteFinger():
    positionNumber = 0
    count = 0
    print('Delete Finger')
    print('Position: ' + str(count))
    while gpio.input(enrol) == True:
        if gpio.input(inc) == False:
            count += 1
            if count > 1000:
                count = 1000
            print('Position: ' + str(count))
            time.sleep(0.2)
        elif gpio.input(dec) == False:
            count -= 1
            if count < 0:
                count = 0
            print('Position: ' + str(count))
            time.sleep(0.2)
    positionNumber = count
    if f.deleteTemplate(positionNumber):
        print('Template deleted!')
        print('Finger Deleted')
        time.sleep(2)

try:
    while True:
        gpio.output(led, HIGH)
        print('Place Finger')
        if gpio.input(enrol) == 0:
            gpio.output(led, LOW)
            enrollFinger()
        elif gpio.input(delet) == 0:
            gpio.output(led, LOW)
            while gpio.input(delet) == 0:
                time.sleep(0.1)
            deleteFinger()
        else:
            searchFinger()

except KeyboardInterrupt:
    print("\nExiting program.")
finally:
    gpio.cleanup()
