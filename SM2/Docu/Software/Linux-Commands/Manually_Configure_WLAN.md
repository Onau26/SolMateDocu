
#### Scan available Networks in the devices vicinity

	sudo iw dev wlan0 scan | egrep 'SSID:|signal:|BSS '
#### Open wpa_supplicant.conf file

	sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
#### Edit wpa_supplicant.conf to match the customers (or previously found) WiFi credentials (don't forget Country-Code)

ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=AT
network={
  ssid="Wartung"
  psk="uHR24wSnmx"
}
#<begin_rpiwifi_entry>
network={
        ssid="customers SSID"
        psk="customers password"
}

#<end_rpiwifi_entry>
#<begin_rpiwifi_entry>


network={
        ssid="YEET"
        psk=12e06092d01b76443e567b99ed721e1cba608d00b97805888405d6f3311922a2
}
#<end_rpiwifi_entry>
#### Make SM connect itself to defined WiFi

	sudo solmatectl control.connect-to-wifi

#### Check in status if SM is connected

	sudo solmatectl status