---
layout: post
title: "Little tricks with adb"
category: howto, android
tags: tricks, howto, android
---

I wanted to use earlier today my hardware device (a Sony Ericsonn Xperia Pro phone), to test some android developments on it.
As it's been a long time since my last development on android, I only had blury memories on how things work.

I bumped on an error, here is how I solved it.

Check the device connection
---------------------------

To connect your phone through USB cable and be able to debug your app on it, you have to follow the steps described on [the official documentation][1]. Basically, it's 
 * **enable USB debugging** on your device, 
 * on Linux, **adding some udev rules** on `/etc/udev/`: for an Sony Ericsson phone, create an executable file with the following content:
 ```shell
    SUBSYSTEM=="usb", ATTR{idVendor}=="0fce", MODE="0666", GROUP="plugdev"
 ```
 * finally, **connect your phone and check** if everything works fine with:
 ```sh
    adb devices
 ```   
 You should have adb in your path to launch the command, otherwise, go to your sdk folder, and launch the `adb` executable from the `platform-tools` subfolder.

Unfortunally, you can get this error when launching the last command:
```sh
List of devices attached
????????????  no permissions
```

How to resolve it
-----------------

If you have a rooted device as I do, you should launch the adb service as root. The service is automatically launch on the first call of adb.

You can launch the following command as root, or as the user who launch adb on the first place:

```sh
adb kill-server
```

Nevertheless, for me this command didn't seem to kill the service, though it was runned without error.
If you are on the same case, you can launch as root:

```shell
killall adb
```

Than, as root, launch:

```bash
adb start-server
```

Finally, when executing again `adb devices`, you should now see your device:

```Bash
List of devices attached
CBZ2J17V0 device
```


[1]: http://developer.android.com/tools/device.html
