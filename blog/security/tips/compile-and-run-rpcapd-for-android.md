# Compile and run rpcapd for Android

Use rpcapd to enable remote network capture.

```bash
me@ubuntu:~$ sudo apt-get install gcc-arm-linux-gnueabi
me@ubuntu:~$ sudo apt-get install byacc
me@ubuntu:~$ sudo apt-get install flex

me@ubuntu:~$ git clone --depth=1 https://github.com/the-tcpdump-group/libpcap.git
me@ubuntu:~$ cd libpcap
me@ubuntu:~/libpcap$ ./configure --host=arm-linux --with-pcap=linux --enable-remote
me@ubuntu:~/libpcap$ make
me@ubuntu:~/libpcap$ cd rpcapd

me@ubuntu:~/libpcap/rpcapd$ file rpcapd
rpcapd: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), dynamically linked, interpreter /lib/ld-, for GNU/Linux 3.2.0, BuildID[sha1]=618e788b4252f72340c370d680407625187a2d9b, with debug_info, not stripped

me@ubuntu:~/libpcap/rpcapd$ arm-linux-gnueabi-gcc -fvisibility=hidden -g -O2 -o rpcapd daemon.o \
    fileconf.o log.o rpcapd.o ../rpcap-protocol.o ../sockutils.o ../fmtutils.o ../sslutils.o ../libpcap.a\
    -lcrypt -lpthread -static

me@ubuntu:~/libpcap/rpcapd$ file rpcapd
rpcapd: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), statically linked, for GNU/Linux 3.2.0, BuildID[sha1]=1828f2fa5576500d254546edb4cc5b6353a9d708, with debug_info, not stripped

me@ubuntu:~/libpcap/rpcapd$ upx rpcapd

me@ubuntu:~/libpcap/rpcapd$ file rpcapd
rpcapd: ELF 32-bit LSB executable, ARM, EABI5 version 1 (GNU/Linux), statically linked, stripped


me@ubuntu:~/libpcap/rpcapd$ adb push rpcapd sdcard

me@ubuntu:~/libpcap/rpcapd$ adb shell


sailfish:/ $ su
sailfish:/ # mv sdcard/rpcapd /data/local/tmp/rpcapd
sailfish:/ # chmod 777 /data/local/tmp/rpcapd

sailfish:/ # /data/local/tmp/rpcapd -h

RPCAPD, a remote packet capture daemon.
Compiled with libpcap version 1.10.0-PRE-GIT (with TPACKET_V3)

USAGE: rpcapd [-b <address>] [-p <port>] [-4] [-l <host_list>] [-a <host,port>]
              [-n] [-v] [-d] [-i] [-D] [-s <config_file>] [-f <config_file>]

  -b <address>    the address to bind to (either numeric or literal).
                  Default: binds to all local IPv4 and IPv6 addresses

  -p <port>       the port to bind to.
                  Default: binds to port 2002

  -4              use only IPv4.
                  Default: use both IPv4 and IPv6 waiting sockets

  -l <host_list>  a file that contains a list of hosts that are allowed
                  to connect to this server (if more than one, list them one
                  per line).
                  We suggest to use literal names (instead of numeric ones)
                  in order to avoid problems with different address families.

  -n              permit NULL authentication (usually used with '-l')

  -a <host,port>  run in active mode when connecting to 'host' on port 'port'
                  In case 'port' is omitted, the default port (2003) is used

  -v              run in active mode only (default: if '-a' is specified, it
                  accepts passive connections as well)

  -d              run in daemon mode (UNIX only) or as a service (Win32 only)
                  Warning (Win32): this switch is provided automatically when
                  the service is started from the control panel

  -i              run in inetd mode (UNIX only)

  -D              log debugging messages

  -s <config_file> save the current configuration to file

  -f <config_file> load the current configuration from file; all switches
                  specified from the command line are ignored

  -h              print this help screen

```

References:

* https://blog.qwerdf.com/2019/03/25/wireshark-with-android/
* https://www.androidtcpdump.com/android-tcpdump/compile

