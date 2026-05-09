# mchose-Keyboard-Fix-for-linux-wake-up-for-3.0-port-only-
exactly what the title says....this is for the turbo and GT series.. it will allow you to use the online hub (given you give your chromimum based browser permissions to see them)  but the extra fetaures like rappy are still being worked on.. this is just a wake script that will survive reboot and power offs..this should work for any keyboard on linux suffering from reconigtion due to "powerdraw"


Create the new wake script:


sudo nano /usr/local/bin/mchose-usb-wake 



#!/usr/bin/env bash
set -u

for d in /sys/bus/usb/devices/*; do
  [ -f "$d/idVendor" ] || continue
  [ -f "$d/idProduct" ] || continue

  if [ "$(cat "$d/idVendor")" = "3837" ] && [ "$(cat "$d/idProduct")" = "3007" ]; then
    echo "Found MCHOSE Ace 68 GT at $d"
    echo 1 > "$d/bConfigurationValue" 2>/dev/null || true
    sleep 1
    echo 1 > "$d/bConfigurationValue" 2>/dev/null || true
    echo "Forced config 1"
  fi
done





Then run this:




chmod +x /usr/local/bin/mchose-usb-wake




Create the new service:



sudo nano /etc/systemd/system/mchose-usb-wake.service 




[Unit]
Description=Wake MCHOSE Ace 68 GT USB config
After=multi-user.target

[Service]
Type=oneshot
ExecStartPre=/usr/bin/sleep 5
ExecStart=/usr/local/bin/mchose-usb-wake

[Install]
WantedBy=multi-user.target




Create the new udev rule:




sudo nano  /etc/udev/rules.d/99-mchose-usb-wake.rules


ACTION=="add", SUBSYSTEM=="usb", ATTR{idVendor}=="3837", ATTR{idProduct}=="3007", TAG+="systemd", ENV{SYSTEMD_WANTS}="mchose-usb-wake.service"




Enable and test:



systemctl daemon-reload
udevadm control --reload-rules
systemctl enable mchose-usb-wake.service
systemctl start mchose-usb-wake.service
lsusb -t



Then check logs:


journalctl -u mchose-usb-wake.service -b --no-pager



reboot


sucess.


if that is giving you issues you can manually run it every reboot with a different kb.

sudo echo 1 > /sys/bus/usb/devices/2-10.3.2/bConfigurationValue

or 

su echo 1 > /sys/bus/usb/devices/2-10.3.2/bConfigurationValue

exit



I hope this helps someone out there.
