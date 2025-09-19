# Packet Sniffing with Wireshark

This guide is for educational purpose only.

## Environment

* Kali Linux running from live USB
* USB WiFi adapter (optional)

## Set Up

Using `airmon-ng` check what processes may interfere with monitoring:

```bash
airmon-ng check wlan0
```

This will list a series of processes that will interfere with WiFi monitoring. You can manually kill them, or using `airmon-ng` to automatically kill them.

```bash
airmon-ng check kill wlan0
```

## Create Monitoring Interface with `iw`

Using the `iw command, create an interface that will be used for monitoring WiFi traffic:

```bash
iw phy phy0 interface add mon0 type monitor
iw dev mon0 set freq 5805
ifconfig wlan0 down
ifconfig mon0 up
```

This will create the `mon0` interface we will be using to sniff traffic with, lock the frequency to 5.808Ghz, take down the regular WiFi interface, and bring up the `mon0` interface.

You can change the channel or frequency the interface is scanning on by issuing similar commands:

```bash
iw dev <devname> set channel <channel>
iw dev <devnmae> set freq <freq>
```

For example:

```bash
iw dev mon0 set channel 161
iw dev mon0 set freq 2442
```

Note: only issue one or the other (freq or channel).

## Start Wireshark

In a root shell execute `wireshark`. Wireshark will launch, and you can select the `mon0` interface and start examining 802.11 frames being broadcasted. If there doesn't seem to be a lot of traffic, use the `iw` command to select another channel or frequency as discussed above.

### Decrypting IEEE 802.11 Frames

In Wireshark open preference with `Ctrl+Shift+P`, navigate to `Protocols` on the left hand side, and fine `IEEE 802.11` in the list. Under the `IEEE 802.11` settings, ensure that `Assume packets have FCS` is checked, and `Ignore the Protection bit` is set to `Yes - without IV`.

Doing this will allow you to see more frames.

## Removing the Monitoring Interface

To remove the manually created monitoring interface and being the regular WiFi interface back up run:

```bash
iw dev mon0 del
ifconfig wlan0 up
```

## Automating it all with `airmon-ng`

The `airmon-ng` script can automate the whole process:

```bash
airmon-ng check kill wlan0
airmon-ng start wlna0
wireshark
```

And then to undo it all:

```bash
airmon-ng stop wlan0mon0
systemctl start NetworkManager wpa_supplicant.service
```
