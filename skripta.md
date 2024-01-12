# Microcontrollers (Arduino, raspberry pi, other...) #

### microcontroler x sbc ###

- microcontrollers
    - run compiled code or at most a precompiled interpereter that runs simple code like micropython or circuitpython

- sbc's (single board controllers) 
    - usually used for more complex things or interfacing multiple microcontrollers with an end user computer
    - do everything a normal computer does but slower
    - usually have lower level i/o

---

### processors ###

- arduino uses mostly atmega chips for their proccessors
    - run compiled c

- raspberry pi uses their own rp2040 cpu
    - run compiled c or an interpreter of your choice so you can run higher level languages without recompiling for every itteration




---

# usecases #
- embeded electronics
- automation
- making things that shouldnt be smart smart anyways

----

# example project #

- single axis raspberry pi gimbal 
    - to demonstrate a simple feedback loop we will build a simple one axis gimbal using an accelerometer as an input to the loop and a servo as an output the accelerometer will be placed on top of the servoarm and will therefore be affected by the servos movement, unless we move the contraption manually everything will stay stationary with the servoarm at 90deg and the accelerometer facing up. Once we move the contraption the accelerometer will measure an undesirable angle and our job is to move the servo according to this information to keep the accelerometer level in this axis.

    - first we have to measure the angle. For this we will use the raspberry pi picos **i2c** interface to connect the accelerometer to the microcontroller. this will give us a simple list of three numbers that from the accelerometer that constantly update

    - next we have to figure out the direction and angle that our accelerometer moved so we can correct for it

    - the last but arguably most important step is to connect the servo to the microcontroller and set one of the digital pins up for **PWM** communication with the servo

    ### wiring ###
    
    - servo
        - 5V    -   5V
        - GND   -   GND
        - S     -   IO Pin 15

    - MPU6050 **imu (inertial measurement unit)**
        - SDA - IO Pin 16
        - SCL - IO Pin 17
        - 5V - 5V 
        - GND - GND

![Alt text][def]

[def]: raspi-pico-pinout.png


# **code** #

- save the imu.py file to the "lib" folder on the pi, if there isnt one just create it

- making the main code

***turn on the led to indicate startup***

```
	LED = machine.Pin("LED", machine.Pin.OUT)
	LED.on()
```


***import all the necesarry libraries***
```
	from imu import MPU6050 //the accelerometer library
	from time import sleep //time
	from machine import Pin, I2C //i2c interface library
    from machine import Pin,PWM //pwm libraray
```

***sellect the correct i2c interface and imu type***
```
	i2c = I2C(0, sda=Pin(0), scl=Pin(1), freq=400000)
	imu = MPU6050(i2c)
```

***define all necessary variables for the servo***

```
    MID = 1500000       //servo PWM center pulse lenght
    currentAngle=MID
    pwm = PWM(Pin(15))  //pin the servo is connected to
    pwm.freq(50)        //PWM frequency (more will use more energy but be smoother)
    pwm.duty_ns(MID)    //set servo to center before startup

```

***red current angle at startup and set it as default***

```
center=round(imu.accel.x,2)

```

***main loop***

```
	while True:
        //save accelerometer and gyro values for each individual axis to variables
	    ax=round(imu.accel.x,2)
	    ay=round(imu.accel.y,2)
	    az=round(imu.accel.z,2)
	    gx=round(imu.gyro.x)
	    gy=round(imu.gyro.y)
	    gz=round(imu.gyro.z)
        
        //print all the values for troubleshooting purposes
        print("ax",ax,"\t","ay",ay,"\t","az",az,"\t","gx",gx,"\t","gy",gy,"\t","gz",gz,"\t","Temperature",tem,"        ",end="\r")         
        
        //insert servo movement part here
        if (ax<center):
            currentAngle++
        if (ax>center):
            currentAngle-1
            
        //wait so we dont overload the servo
	    sleep(0.2) 
```
