# Klipper-OPiZ2W
Trying to get my OrangePi Zero2W to run Klipper/Mainsail to run a 3D printer. 
(this will be my first attempt at a write-up like this, so be kind... or ruthless. I don't care. Either way I'll do the best I can.)

## Finding a compatible OS Image
   This was harder than expected. My first thought was to turn to the official [Zero2W page on OrangePi.org](http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/service-and-support/Orange-Pi-Zero-2W.html). Through (much) trial and error I found that only one worked for me: **"[Orangepizero2w_1.0.0_debian_bullseye_server_linux5.4.125](https://drive.google.com/file/d/1DfEyIZBMi8tDQ61AEQEgoNxHBhoK1lLL/view?usp=drive_link)"** Direct link to Google Drive where image is stored [here.](https://drive.google.com/drive/folders/1X_OFH8EqGqRzvlz4OZpzcH6wqk9-tAja)

I'm also using the [expansion board](#expansion-board-hardware) for the Z2W, the one with the Ethernet port, 2xUSBs (2.0), 100M ethernet jack, two GPIO buttons, an infrared reader and a POWER BUTTON. (this comes in handy, believe it or not)

Generic steps: 
1. Flash the downloaded image to an SD card. OrangePi recommends the SanDisk brand, class 10 or better for reliability & longevity.
2. Go get tea.
3. Put SD card into OPi Z2W. Plug in Ethernet, plug in power. At this stage, there should be two solid red LEDs; one on the main board, and one on the expansion board.  
   1. If you *are not* using the expansion board, just wait. Eventually the green LED *should* start blinking alongside the red one. This is good, it means it has booted.  
   2. If you (like me) *are* using the expansion board, it won't do anything until you **hit the power button.** Hold it for 1–2 seconds, then let go and wait.  
      - It will take ~5 minutes or more for the first boot, so be patient.  
      - Once it boots properly:
        - The red LED should be solid.
        - The green LED blinks in twos.
        - The Ethernet port should light up eventually as well.
5. Open [Advanced IP Scanner](https://www.advanced-ip-scanner.com/) and refresh the list.
6. Look for the hostname **orangepizero2w**, if you can't find it here, go to your router's client list. Copy this IP.
7. Open your SSH tool of choice (such as [MobaXterm](https://mobaxterm.mobatek.net/), [Putty](https://www.putty.org/)) and start an SSH session with the IP you just copied.
8. Login as *root* with the default password being *orangepi*. I strongly suggest you change this after first login, for security purposes.
   1. Use the command `passwd` when logged in to prompt the password change.
9. Create new user. I chose "opi". `sudo adduser opi` Set password, run the following command to add your new user to the `sudoers` group `sudo usermod -aG sudo opi` afterwards, end terminal session. 
10. Start new terminal session with your OrangePi Z2W with the new user+password combination.
11. Next I'll be installing [KIAUH.](https://github.com/dw-0/kiauh)
    1. Update & install Git. `sudo apt-get update && sudo apt-get install git -y`
    2. Download KIAUH to home directory. `cd ~ && git clone https://github.com/dw-0/kiauh.git`
    3. Start KIAUH. `./kiauh/kiauh.sh`
    4. Proceed with installing all the desired aspects of Klipper et al.

/end night 1 effort   

12. Now that Klipper is setup on the OPi Z2W, I wanted to set up the WiFi connection so I don't *have* to have the OPi plugged in via ethernet.
   1. I used this command to scan the network list: `nmcli device wifi list`
   2. Then I used this to connect to my desired WiFi network: `nmcli device wifi connect "<SSID>" password "<yourpassword>"`
      1. *Note: I was only able to connect to a network only running 2.4GHz*
   3. I confirmed that it was working with: `ip a | grep wlan`, copy this IP. You'll need it for the next SSH session. You can do a ping if you want to: `ping -c 3 google.com`
   4. Wifi Setup is complete!

13. Next, I unplugged the ethernet, and navigated to the IP address via web browser to ensure the Mainsail GUI would still connect.
14. For the next few steps, I found [this great video by Vector 3D on YouTube](https://www.youtube.com/watch?v=N41JY1Gukuk) that went through step-by step how to connect Klipper on the Pi to the control board. I'm using a 4.2.2 board on an Ender 3v2, as well as a 1.1.3 in an original Ender.
15. Start an SSH session as the non-`root` (`opi`) user. This is where it breaks into two lanes; one for the 4.2.2. board (which I'll be doing first) and the second for the 1.1.3.
16. ### 4.2.2
      1. Change directory to klipper `cd ./klipper/`
      2. Initiate the make menu interface `make menuconfig`
      3. Set these settings to:

            `-Enable extra low-level configuration options`
            `-Micro-controller Architecture (STMicroelectronics STM32)`
            `Processor Model (STM32F103)           # Should already be set correctly`
            `Bootloader offset (28KiB bootloader)`
            `Communication interface (Serial (on USART1 PA10/PA9))`
         
       5. Open MobaXterm and use the file browser feature to navigate to `/home/user/klipper/out` and grab the `klipper.bin` file, save it to your computer.
       6. Copy the `klipper.bin` file to the root of an empty SD card, and pop it into the printer (Ender3v2) and restart the printer.
       7. Go back to MobaXterm, and issue the following command to get your serial ID: `ls /dev/serial/by-id/*`
       8. Copy this ID. Go to the Mainsail interface (or Fluidd), locate `printer.cfg` and find the `[mcu]` section, and replace the serial ID there with the one you just copied. Save & restart. 
       9. Find the example config from [This GH repository](https://github.com/Klipper3d/klipper/blob/master/config/printer-creality-ender3-v2-2020.cfg) and then edit it to your requirements. I have a BL touch installed, and made some other changes. I'll have the config file uploaded here.
          1. *Tip: Use the `BLTOUCH_DEBUG COMMAND=pin_up` and `BLTOUCH_DEBUG COMMAND=pin_down` commands to test the BL Touch before attempting to home the printer.*
          2. *Tip 2: If you can't get the BL Touch to work, check to see if your board uses a pullup resistor for the signal, and use a `^` before the pin number; eg. `^PB1`*
      11. 


## Handy Info
### Hardware features of Orange Pi Zero 2w
| Feature               | Specification                                                                 |
|-----------------------|---------------------------------------------------------------------------------|
| **CPU**               | Allwinner H618 quad-core 64-bit 1.5GHz Cortex-A53                              |
| **GPU**               | Mali G31 MP2<br>Supports OpenGL ES 1.0/2.0/3.2, OpenCL 2.0                      |
| **Memory**            | 1GB / 1.5GB / 2GB / 4GB LPDDR4 (shared with GPU)                                |
| **Onboard Storage**   | TF card slot, 16MB SPI Flash                                                    |
| **WiFi + Bluetooth**  | 20U5622 chip<br>Supports IEEE 802.11 a/b/g/n/ac, Bluetooth 5.0                 |
| **Video Output**      | Mini HDMI 2.0                                                                   |
| **Audio Output**      | Mini HDMI audio                                                                |
| **Power Supply**      | Type-C 5V/2A                                                                    |
| **USB Ports**         | 2 × Type-C USB 2.0                                                              |
| **40-Pin Expansion**  | GPIO, UART, I²C, SPI, PWM                                                       |
| **24-Pin Expansion**  | USB 2.0 ×2, 100M Ethernet, IR input, audio out, TV-OUT, power & LRADC buttons  |
| **LED Indicators**    | Power LED and Status LED                                                       |
| **Supported OS**      | Android 12 TV, Debian 11/12, Ubuntu 20.04/22.04, Orange Pi OS (Arch)           |

---

### Physical Specifications

| Parameter         | Value              |
|------------------|--------------------|
| **PCB Size**      | 30mm × 65mm × 1.2mm |
| **Weight**        | 12.5g              |

<p align="center">
  <img src="https://github.com/user-attachments/assets/a30af888-83de-476c-b8e4-afd9ec9463f3" width="60%" alt="0825-zero2w-img01" />
</p>

### Expansion Board Hardware
| Feature             | Specification                                      |
|---------------------|----------------------------------------------------|
| **Ethernet Port**    | 100 Mbps                                           |
| **USB**              | USB 2.0 × 2                                        |
| **Keys**             | 1× Power On/Off key, 2× Custom keys                |
| **Audio**            | 3.5mm headphone jack (audio + TV-out combined)     |
| **Infrared Receiver**| 1× Infrared receiver tube                         |
| **PCB Dimensions**   | 30mm × 65mm                                        |

<p align="center">
  <img src="https://github.com/user-attachments/assets/bc87381c-730d-4576-afab-274a4f307019" width="60%" alt="0825-zero2w-img01" />
</p>

# References
1. [OrangePi Wiki](http://www.orangepi.org/orangepiwiki/index.php/Orange_Pi_Zero_2W)


