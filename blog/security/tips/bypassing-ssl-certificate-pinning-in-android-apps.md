# Bypassing SSL certificate pinning in Android apps

Let's say you want to know what is this Android application doing behind their user interface. What are the API endpoints that they call, what is the request and response format, etc.  
  
So you launch Burp Suite, setup a proxy, but alas! You only got SSL certificate unknown error! Most likely this is caused by SSL certificate pinning in the application.  
  
So what to do now? Luckily there is a script running via Frida that can be utilized to bypass this SSL pinning thing:  
  
1. Download Frida server to Android device

```bash
$ wget https://github.com/frida/frida/releases/download/12.4.4/frida-server-12.4.4-android-arm64.xz
$ xz -d frida-server-12.4.4-android-arm64.xz
$ adb push frida-server-12.4.4-android-arm64 /data/local/tmp/
```

2. Run Frida server

```bash
$ adb shell

> su
> chmod +x /data/local/tmp/frida-server-12.4.4-android-arm64
> /data/local/tmp/frida-server-12.4.4-android-arm64
```

3. Open the targeted app in Android device  
4. Bypass SSL pinning by using a bypass script via Frida

```bash
$ pip install frida-tools
$ frida --codeshare sowdust/universal-android-ssl-pinning-bypass-2 --no-pause -U com.example.app
```

5. Continue using the app in android device, SSL-pinning protected endpoint should be able to be captured via proxy now  


