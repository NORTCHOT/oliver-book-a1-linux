# Onda Oliver Book A1 stuff for Arch Linux

Guides, configs, and other files needed for all components of this device to work properly on Arch Linux. This list will be updated as needed.


## How to rotate the framebuffer console during boot? (boot logs)
Simply add `fbcon=rotate:1` to your boot options, where `1` is the rotation angle (1 - 90, 2 - 180, 3 - 270).  
This setting only affects the framebuffer console (TTY), so you'll need to configure screen rotation within the system itself through your DE.

## How to make the touchscreen work?
The touchscreen driver is usually already included in the kernel. You can check that with `sudo dmesg | grep silead`. If you see something like this:  
```
[    9.137386] silead_ts i2c-MSSL1680:00: Silead chip ID: 0x80360000
[    9.279035] silead_ts i2c-MSSL1680:00: Direct firmware load for silead/mssl1680.fw failed with error -2
[    9.279046] silead_ts i2c-MSSL1680:00: Firmware request error -2
[    9.405250] silead_ts: probe of i2c-MSSL1680:00 failed with error -2
```
you need the firmware, which you can download [here](https://raw.githubusercontent.com/NORTCHOT/oliver-book-a1-linux/refs/heads/main/mssl1680.fw). Then move the file to the `/lib/firmware/silead/` directory.  
After that you need to add these parameters to your boot options:  
`i2c_touchscreen_props=MSSL1680:touchscreen-min-x=4:touchscreen-min-y=3:touchscreen-size-x=1981:touchscreen-size-y=1529:touchscreen-swapped-x-y:silead,home-button`  
That's it. Just reboot and touchscreen should work properly.

## Accelerometer
You need to install the `iio-sensor-proxy` daemon first:  
```bash
sudo pacman -S iio-sensor-proxy
sudo systemctl enable iio-sensor-proxy --now
```
Check your DE settings for auto-rotate enabled and try rotating the device. It should work, but you need to create a udev rule for the right orientations. Make the `/etc/udev/hwdb.d/99-accelerometer.hwdb` file:  
```
sensor:modalias:*
  ACCEL_MOUNT_MATRIX=0, 1, 0; 1, 0, 0; 0, 0, 1
```
Note that there are two spaces before `ACCEL_MOUNT_MATRIX=0, 1, 0; 1, 0, 0; 0, 0, 1`. That's important.  
After this:  
```bash
sudo systemd-hwdb update
sudo udevadm trigger
sudo systemctl restart iio-sensor-proxy
```
That's all.

---

Everything else should usually work fine. But if you've encountered some other issues, know how to fix something else or see a mistake in that guide - PRs are open.

## Known issues
**1. Touchscreen taps don't quite match up with the actual ones**  
Most likely, you can get better results by playing around with the min-x/y and size-x/y values. But I'm too lazy :D  
**2. Diagonal swipes on the touchscreen follow a "ladder" pattern**  
Yeah, I noticed that too, but haven't found the fix yet. Maybe someone knows how to fix it?  
**3. Sound not working**  
Same as 2.

