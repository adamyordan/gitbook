# Capturing Specific App's Network Traffic in Android without Proxy

There maybe times when you want to know the network activity of an application in your mobile device. Yes, you can just use proxy \(Burp suite proxy is super!\), but sometimes they may introduce some issues, e.g. SSL pinning error, network proxy detection, and they cannot capture non-standard HTTP packets \(e.g. UDP packets\).

### Installing TCPDump in Android

```text
$ wget https://www.androidtcpdump.com/download/4.9.2/tcpdump
$ adb push tcpdump /sdcard/tcpdump
$ adb shell
android$ su
android# mount -o rw,remount /system
android# mv /sdcard/tcpdump /system/xbin/tcpdump
android# chmod 777 /system/xbin/tcpdump
```

### Capture all network traffic in Android device

This will capture all network traffic, not specific to a single app. Run with Root.

```text
android# tcpdump -i any -s 0 -w /sdcard/capture.pcap
```

### Capture specific app's network traffic in Android device

Since network stream / packets did not have information about PID or UID, capturing network traffic for a specific process is not simple. Fortunately, in Android, every installed app is assigned unique UID, so we can capture packets sent by a specific UID over the network interfaces.

```text
# get APP_UID in this file, e.g. 10303
android# cat /data/system/packages.list | grep com.example.app

# use iptables to put the outgoing traffic from the app in an NFLOG group
android# iptables -A OUTPUT -m owner --uid-owner APP_UID -j NFLOG --nflog-group 30

# use tcpdump to capture packets from the group
android# tcpdump -i nflog:30 -s 0 -w /sdcard/capture.pcap
```

However, this will only capture outgoing/outbound traffic, incoming/inbound traffic is not captured.

