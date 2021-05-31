# Forwarding TCPDump from Android to Wireshark

Continuation from [previous post](capturing-specific-apps-network-traffic-in-android-without-proxy.md). We want a realtime capture of network traffic of our Android app.

```text
laptop$ adb forward tcp:31337 tcp:31337

laptop$ adb forward --list
FA69K0312058 tcp:31337 tcp:31337

laptop$ adb shell

android$ su
android# tcpdump -i nflog:30 -s 0 -w - -nS | nc -l -p 31337

laptop$ nc localhost 31337 | wireshark -i - -kS
```

