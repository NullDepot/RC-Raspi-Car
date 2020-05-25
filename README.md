# RC-Raspi-Car
The Raspi-Car project is the creation of a remote-controlled car that can be operated using a headless Raspberry Pi set up. This project takes advantage of a live video stream using MJPG Streamer in order to control the car over the network without needing to maintain a line of sight on the car itself.

The following modules were used for this project:

* Raspberry Pi 3
* CamJam EduKit #3 - Robotics (additional resources can be found here: https://github.com/CamJam-EduKit/EduKit3)
* A custom built chassis (a simple cardboard box will also suffice. See more in the CamJam-EduKit link above)
* ZeroCam Fisheye NightVision - for PiZero & Raspberry Pi 3
* A powerbank for portable power

Once all of the modules have been assembled, you are ready to follow the instructions below.

---
# Clone this repository

Make sure git is installed on your Raspberry Pi first
```
$ sudo apt update
$ sudo apt full-upgrade
$ sudo apt install git
```

Then you can clone this repo
```
$ git clone https://github.com/NullDepot/RC-Raspi-Car.git
```
---
# Configure the camera module

1. Enable camera support
```
$ sudo raspi-config
```
Use the cursor keys to select and open *Interfacing Options*. Select *Camera* and follow the prompt to enable the camera.

2. Test the camera
```
raspistill -v -o test.jpg
```
An image should have been saved entitled `test.jpg`. Feel free to remove this once you confirm your camera works.

---
# Control the car using keyboard inputs

1. Run the script included in this repo
```
$ cd ./RC-Raspi-Car
$ python rc_keyboard.py
```

The following keystrokes can be used:
* W - forward
* S - backward
* A - left
* D - right
* Q - quit

In the interest of easy access, I recommend copying the `rc_keyboard.py` file to your home directory.
```
$ cp ./rc_keyboard.py ~/rc_keyboard.py
```
The script can now be run as soon as you connect to your pi through ssh with the `$ python rc_keyboard.py` command.

---
# Setting up MJPG-Streamer

1. Clone the GitHub repo by jacksonliam
```
$ cd ~
$ git clone https://github.com/jacksonliam/mjpg-streamer.git
$ cd mjpg-streamer/
```

2. Install the dependencies
```
sudo apt-get install cmake libjpeg8-dev
```

3. Set up MJPG Streamer
```
$ cd mjpg-streamer-experimental
$ make
$ sudo make install
```

4. Usage
```
$ export LD_LIBRARY_PATH=.
$ mjpg_streamer -i 'input_uvc.so -d /dev/video0 -r 1280x720 -f 15 -n' -o 'output_http.so -w www -p 8080'
```
You can check if your stream works by going to `http://<IP-address>:8080` in your web-browser.

Really, your car is good to go at this point. However, we want to edit our `rc.local` file so that MJPG Streamer starts automatically once the car turns on.


5. Run MJPG at boot

Edit your `/etc/rc.local` file and add the following lines to the bottom of the file before `exit 0`. These are the same commands that we ran in the previous step, except we are giving the path to the MJPG Streamer folder.
```
## Run MJPG Streamer
export LD_LIBRARY_PATH=/home/pi/mjpg-streamer/mjpg-streamer-experimental
/home/pi/mjpg-streamer/mjpg-streamer-experimental/mjpg_streamer -i 'input_uvc.so -d /dev/video0 -r 1280x720 -f 15 -n' -o 'output_http.so -w /home/pi/mjpg-streamer/mjpg-streamer-experimental/www -p 8080'
```
Save and exit the file, then reboot your pi. MJPG streamer should now be running on startup.


6. Stream your video!

That's everything you need to get the stream running. You can go back to the site the same was as you did in step 4, at `http://<IP-address>:8080` which should be running already. Run the `rc_keyboard.py` script to control your car and watch through the stream.

---
