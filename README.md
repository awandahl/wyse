# Wyse NEW

#### /etc/udev/rules.d

99-radio-devices.rules
```
# QMX Radio
ACTION=="add", SUBSYSTEM=="tty", ATTRS{idVendor}=="0483", ATTRS{idProduct}=="a34c", SYMLINK+="ttyQMX", MODE="0666", GROUP="dialout"

# Winkeyer Large Keyer
# ACTION=="add", SUBSYSTEM=="tty", ATTRS{idVendor}=="1a86", ATTRS{idProduct}=="7523", SYMLINK+="ttyWinkeyer", MODE="0666", GROUP="dialout"

# K3NG keyer for cwdaemon
ACTION=="add", SUBSYSTEM=="tty", ATTRS{idVendor}=="1a86", ATTRS{idProduct}=="7523", SYMLINK+="ttyCW", MODE="0666", GROUP="dialout"

# Elecraft K2
ACTION=="add", SUBSYSTEM=="tty", ATTRS{idVendor}=="0403", ATTRS{idProduct}=="6001", SYMLINK+="ttyK2", MODE="0666", GROUP="dialout"
```

#### /etc/systemd/system

rigctld-k2.service
```
[Unit]
Description=Rigctld for Elecraft K2 Radio
After=network.target

[Service]
User=aw
Group=aw
ExecStart=/usr/local/bin/rigctld \
  -m 2021 \
  -r /dev/ttyK2 \
  -s 4800 \
  -t 4532
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

rigctld-qmx.service
```
[Unit]
Description=Rigctld for QMX Radio
After=network.target

[Service]
User=aw
Group=aw
ExecStart=/usr/local/bin/rigctld \
# TS480
#  -m 2028 \
# QMX latest
#  -m 2057 \
# QCX/QDX
  -m 2052 \
# TS440
#  -m 2002 \
  -r /dev/ttyQMX \
  -s 115200 \
  -t 4533
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

cwdaemon.service (default port 6789)
```
[Unit]
Description=CWdaemon service for keying
After=dev-ttyS0.device
Requires=dev-ttyS0.device

[Service]
Type=forking
User=aw
Group=aw
ExecStart=/usr/sbin/cwdaemon -d ttyS0 -x n
Restart=always
RestartSec=5
Environment="PATH=/usr/local/bin:/usr/bin:/bin:/usr/sbin"

[Install]
WantedBy=multi-user.target
```

# systemctl

### Basic systemctl Commands

| Command | Description |
| :-- | :-- |
| systemctl status [service] | See status of a service (running, errors, logs) |
| systemctl start [service] | Start a service NOW |
| systemctl stop [service] | Stop a service |
| systemctl restart [service] | Restart (stop \& start) a service |
| systemctl reload [service] | Reload config of a service (if supported) |
| systemctl enable [service] | Enable service to start at boot |
| systemctl disable [service] | Disable service from starting at boot |
| systemctl is-active [service] | Is the service running now? |
| systemctl is-enabled [service] | Is the service enabled to autostart at boot? |
| systemctl list-units --type=service | List all currently loaded/running services |
| systemctl list-unit-files --type=service | List all services (enabled/disabled/etc) |

### System Power Commands

| Command | Description |
| :-- | :-- |
| systemctl reboot | Reboot the system |
| systemctl poweroff | Shut down the system |
| systemctl suspend | Suspend (sleep) the system |
| systemctl hibernate | Hibernate the system |
| systemctl halt | Halt the system |

### System State / Target Commands

| Command | Description |
| :-- | :-- |
| systemctl get-default | Show default system target/runlevel |
| systemctl set-default X | Change default target (e.g. multi-user.target, graphical.target) |
| systemctl isolate X.target | Switch to a different system state |
| systemctl rescue | Enter rescue (single-user) mode |

# wyse OLD

### systemd winkeyerserial
Install winkeyerserial to system path.

```
[Unit]
Description=PyWinKeyerSerial
After=network.target

[Service]
User=aw
Group=aw
ExecStart=/usr/local/bin/winkeyerserial  # Install to system path
Restart=always
RestartSec=5
Environment="QT_QPA_PLATFORM=offscreen"
Environment="PATH=/usr/local/bin:/usr/bin:/bin"

[Install]
WantedBy=multi-user.target
```
### systemd Elecraft K2
```
[Unit]
Description=Rigctld for Elecraft K2 Radio
After=network.target

[Service]
User=aw
Group=aw
ExecStart=/usr/local/bin/rigctld \
  -m 2021 \
  -r /dev/ttyK2 \
  -s 4800 \
  -t 4532
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target

```
### systemd QMX
```
[Unit]
Description=Rigctld for QMX Radio
After=network.target

[Service]
User=aw
Group=aw
ExecStart=/usr/local/bin/rigctld \
  -m 2057 \
  -r /dev/ttyQMX \
  -s 115200 \
  -t 4533
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target


```




```
### QMX


aw@solar:~$ udevadm info -a -n /dev/ttyACM0 | grep -E "idVendor|idProduct|KERNELS"
    KERNELS=="1-3:1.0"
    KERNELS=="1-3"
    ATTRS{idProduct}=="a34c"
    ATTRS{idVendor}=="0483"
    KERNELS=="usb1"
    ATTRS{idProduct}=="0002"
    ATTRS{idVendor}=="1d6b"
    KERNELS=="0000:00:15.0"
    ATTRS{dbc_idProduct}=="0010"
    ATTRS{dbc_idVendor}=="1d6b"
    KERNELS=="pci0000:00"
    
### Winkeyer Large

aw@solar:~$ udevadm info -a -n /dev/ttyUSB0 | grep -E "idVendor|idProduct|KERNELS"
    KERNELS=="ttyUSB0"
    KERNELS=="1-6.2:1.0"
    KERNELS=="1-6.2"
    ATTRS{idProduct}=="7523"
    ATTRS{idVendor}=="1a86"
    KERNELS=="1-6"
    ATTRS{idProduct}=="5415"
    ATTRS{idVendor}=="0bda"
    KERNELS=="usb1"
    ATTRS{idProduct}=="0002"
    ATTRS{idVendor}=="1d6b"
    KERNELS=="0000:00:15.0"
    ATTRS{dbc_idProduct}=="0010"
    ATTRS{dbc_idVendor}=="1d6b"
    KERNELS=="pci0000:00"
```


## **1. Identify Devices and Ports**

### **Step 1: Find `idVendor`, `idProduct`, and Physical Port Path**

For each device:

1. Plug the device into the desired USB port.
2. Run:

```bash
udevadm info -a -n /dev/ttyACM0 | grep -E "idVendor|idProduct|KERNELS"
```

Example output:

```
ATTRS{idVendor}=="0483"
ATTRS{idProduct}=="a34c"
KERNELS=="1-3"  # Physical port path (e.g., bus 1, port 3)
```


Repeat for all devices, noting their `idVendor`, `idProduct`, and `KERNELS` (port path).

---

## **2. Create Persistent Udev Rules**

### **Step 2: Define udev Rules for Each Device**

Create a file for each device in `/etc/udev/rules.d/` (e.g., `99-radio.rules`, `99-arduino.rules`):

```bash
sudo nano /etc/udev/rules.d/99-radio.rules
```

Add rules like this:

```bash
# QMX Radio (Port 1-3)
ACTION=="add", SUBSYSTEM=="tty", KERNELS=="1-3", ATTRS{idVendor}=="0483", ATTRS{idProduct}=="a34c", SYMLINK+="ttyQMX", MODE="0666", GROUP="dialout"

# Arduino (Port 1-4)
ACTION=="add", SUBSYSTEM=="tty", KERNELS=="1-4", ATTRS{idVendor}=="2341", ATTRS{idProduct}=="0043", SYMLINK+="ttyArduino", MODE="0666", GROUP="dialout"

# Pi Pico (Port 2-1)
ACTION=="add", SUBSYSTEM=="tty", KERNELS=="2-1", ATTRS{idVendor}=="2e8a", ATTRS{idProduct}=="000a", SYMLINK+="ttyPico", MODE="0666", GROUP="dialout"
```


### **Step 3: Reload Udev Rules**

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```

---

## **3. Create Systemd Services**

### **Step 4: Define Templated Systemd Service Files**

Create a template service file (e.g., `/etc/systemd/system/rigctld@.service`):

```bash
sudo nano /etc/systemd/system/rigctld@.service
```

Add:

```ini
[Unit]
Description=Rigctld for %i
BindsTo=dev-%i.device
After=dev-%i.device

[Service]
ExecStart=/usr/local/bin/rigctld -m 2057 -r /dev/%i -t 4533 -s 115200
Restart=on-failure

[Install]
WantedBy=multi-user.target
```


### **Step 5: Enable Services for Each Device**

```bash
sudo systemctl enable rigctld@ttyQMX
sudo systemctl enable rigctld@ttyArduino
sudo systemctl enable rigctld@ttyPico
```

---

## **4. Verify and Troubleshoot**

### **Step 6: Check Symlinks and Services**

- Confirm symlinks exist:

```bash
ls -l /dev/ttyQMX /dev/ttyArduino /dev/ttyPico
```

- Check service status:

```bash
systemctl status rigctld@ttyQMX
journalctl -u rigctld@ttyQMX
```


### **Step 7: Test Communication**

Use `minicom` or `screen` to verify device access:

```bash
sudo minicom -D /dev/ttyQMX -b 115200
```

---

## **5. Advanced Configuration**

### **Handling Multiple Instances**

- **Unique Ports**: Assign each device to a dedicated physical USB port.
- **Service Ports**: Use different TCP ports (e.g., `4533`, `4534`) for each `rigctld` instance.


### **Auto-Start on Plug-In**

Use `udev` rules to trigger services dynamically:

```bash
ACTION=="add", SUBSYSTEM=="tty", ..., TAG+="systemd", ENV{SYSTEMD_WANTS}="rigctld@%k.service"
```

---

## **Final Notes**

- **Physical Port Binding**: Use `KERNELS` in udev rules to tie devices to specific ports, avoiding conflicts.
- **Permissions**: Ensure `MODE="0666"` and `GROUP="dialout"` for non-root access.
- **Scalability**: Use systemd templates to manage multiple devices with minimal configuration.

By following this workflow, you’ll have a robust setup where each device is persistently mapped to its designated port and managed by dedicated services. Let me know if you need further refinements!

<div>⁂</div>

[^1]: https://acroname.com/blog/how-find-vendor-id-and-product-id-your-usb-device

[^2]: https://unix.stackexchange.com/questions/566682/get-the-device-path-of-usb-given-device-id

[^3]: https://wiki.domoticz.com/PersistentUSBDevices

[^4]: https://www.derekdemuro.com/2020/06/06/finding-usb-devices-with-lsusb-idvendor-idproduct/

[^5]: https://itsfoss.com/list-usb-devices-linux/

[^6]: https://unix.stackexchange.com/questions/346259/how-to-get-a-usb-device-path-by-its-idvendor-and-idproduct

[^7]: https://askubuntu.com/questions/349995/find-usb-devices-directory-sys-bus-usb-devices-using-idvendor-idproduct

[^8]: https://stackoverflow.com/questions/70513926/how-to-get-the-vendor-and-product-id-of-a-usb-device-from-sys-bus-devices-usb-d

[^9]: https://askubuntu.com/questions/987966/how-do-i-get-the-usb-drive-path-and-label-by-vendor-id-and-product-id

[^10]: https://askubuntu.com/questions/56938/getting-information-on-my-usb-devices

[^11]: https://www.baeldung.com/linux/check-for-usb-devices

[^12]: https://stackoverflow.com/questions/3279800/how-to-get-usb-vendor-and-product-info-programmatically-on-linux

[^13]: https://stackoverflow.com/questions/5565848/how-to-find-the-base-address-of-usb-to-parallel-port-device-in-linux

[^14]: https://www.reddit.com/r/linuxquestions/comments/4m6l1j/multiple_devices_with_same_product_id_vendor_id/

[^15]: https://unix.stackexchange.com/questions/747084/does-every-usb-device-have-a-vendor-id-and-product-id

[^16]: https://www.linuxquestions.org/questions/linux-newbie-8/how-to-detect-usb-devices-like-mp3-players-571705/

[^17]: https://superuser.com/questions/983399/how-to-uniquely-identify-a-usb-device-in-linux

[^18]: https://superuser.com/questions/645459/does-xubuntu-keep-a-log-of-all-usb-devices-ever-connected-to-the-system

[^19]: https://wiki.debian.org/HowToIdentifyADevice/USB

[^20]: https://itsfoss.com/find-mac-address-linux/


Here’s how to configure your `systemd` services to run as the user `aw`:

---

### **1. Modify the Service Files**

For each service file (e.g., `rigctld-qmx.service`, `rigctld-winkeyer.service`), add or modify the following entries:

```ini
[Service]
User=aw
Group=aw
```


#### Example: `/etc/systemd/system/rigctld-qmx.service`

```ini
[Unit]
Description=Rigctld for QMX Radio
After=network.target

[Service]
User=aw
Group=aw
ExecStart=/usr/local/bin/rigctld \
  -m 2057 \
  -r /dev/ttyQMX \
  -s 115200 \
  -t 4533
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

---

### **2. Ensure User Has Necessary Permissions**

The user `aw` must have access to the serial devices (`/dev/tty*`) and any symlinks created by udev.

#### Add `aw` to the `dialout` Group

The `dialout` group typically has access to serial devices. Add the user `aw` to this group:

```bash
sudo usermod -a -G dialout aw
```

Log out and log back in for the group membership changes to take effect.

---

### **3. Reload and Restart Services**

After modifying the service files, reload the systemd configuration and restart the services:

```bash
sudo systemctl daemon-reload

# Restart services as user "aw"
sudo systemctl restart rigctld-qmx.service
sudo systemctl restart rigctld-winkeyer.service
```

---

### **4. Verify Services Are Running as User `aw`**

Check the status of a service to confirm it is running under the correct user:

```bash
systemctl status rigctld-qmx.service
```

Look for a line like this:

```
Main PID: 1234 (rigctld)
Tasks: 1 (limit: 4678)
Memory: 2.3M
CGroup: /system.slice/rigctld-qmx.service
         └─1234 /usr/local/bin/rigctld -m 2057 -r /dev/ttyQMX -s 115200 -t 4533

User=aw
Group=aw
```

---

### **5. Optional: Run User Services in User Context**

If you prefer, you can create **user-specific systemd services** that run in the user's context instead of globally. These services are managed by the user and do not require `sudo`.

#### Step 1: Create a User Service Directory

Create a directory for user-specific services:

```bash
mkdir -p ~/.config/systemd/user/
```


#### Step 2: Add a Service File for Each Device

For example, create a service for QMX:

```bash
nano ~/.config/systemd/user/rigctld-qmx.service
```

Add:

```ini
[Unit]
Description=Rigctld for QMX Radio

[Service]
ExecStart=/usr/local/bin/rigctld \
  -m 2057 \
  -r /dev/ttyQMX \
  -s 115200 \
  -t 4533

[Install]
WantedBy=default.target
```


#### Step 3: Enable and Start User Services

Reload systemd for the user and enable/start the service:

```bash
systemctl --user daemon-reload

# Enable and start service for QMX radio
systemctl --user enable rigctld-qmx.service
systemctl --user start rigctld-qmx.service

# Check status of service
systemctl --user status rigctld-qmx.service
```


#### Step 4: Enable User Services at Boot (Optional)

To ensure user services start at boot, enable lingering for the user:

```bash
sudo loginctl enable-linger aw
```

This allows user services to start even if no one is logged in.

---

### **Summary**

- To run services as `aw`, add `User=aw` and `Group=aw` in your service files.
- Ensure `aw` has access to serial devices by adding it to the `dialout` group.
- Optionally, use **user-specific systemd services** if you prefer managing them in the user's context.

This setup ensures your services are running with minimal privileges while maintaining proper functionality. Let me know if you need further assistance!

