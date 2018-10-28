# Initializing Raspberry Pi

## References

* [http://blog.xcoda.net/82](http://blog.xcoda.net/82) 
* [https://m.blog.naver.com/lovespreads/221179834970](https://m.blog.naver.com/lovespreads/221179834970)
* [Raspberry pi setting OTG](https://medium.com/@aallan/setting-up-a-headless-raspberry-pi-zero-3ded0b83f274)
* [Bluetooth communication between raspberry pi and arduino](http://webnautes.tistory.com/979)
* [Bluetooth pairing](https://www.cnet.com/how-to/how-to-setup-bluetooth-on-a-raspberry-pi-3/)
* * * * 
## Logs

### Bluetooth manipulation on Raspberry Pi

```bash
$ sudo apt-get install bluetooth bluez bluez-tools
$ sudo bluetoothctl
```

{% hint style="info" %}
$ means a writable line in Bash\(Terminal\). type the commands just after $ signs.

Likewise, \# means a writable line in "bluetooth" program. type the commands just after \* signs.
{% endhint %}

```bash
[bluetooth]# agent on
Agent registered
[bluetooth]# default-agent
Default agent request successful
[bluetooth]# scan on
Discovery started
[NEW] Device 00:18:E4:34:D1:C9 00-18-E4-34-D1-C9
[CHG] Device 00:18:E4:34:D1:C9 LegacyPairing: no
[CHG] Device 00:18:E4:34:D1:C9 Name: HC-06
[CHG] Device 00:18:E4:34:D1:C9 Alias: HC-06
[CHG] Device 00:18:E4:34:D1:C9 LegacyPairing: yes
[bluetooth]# pair 00:18:E4:34:D1:C9
Attempting to pair with 00:18:E4:34:D1:C9
[CHG] Device 00:18:E4:34:D1:C9 Connected: yes
Request PIN code
[agent] Enter PIN code: 1234
[CHG] Device 00:18:E4:34:D1:C9 UUIDs: 00001101-0000-1000-8000-00805f9b34fb
[CHG] Device 00:18:E4:34:D1:C9 ServicesResolved: yes
[CHG] Device 00:18:E4:34:D1:C9 Paired: yes
Pairing successful
[CHG] Device 00:18:E4:34:D1:C9 ServicesResolved: no
[CHG] Device 00:18:E4:34:D1:C9 Connected: no
[CHG] Device 00:18:E4:34:D1:C9 RSSI: -58
[bluetooth]# connect 00:18:E4:34:D1:C9
Attempting to connect to 00:18:E4:34:D1:C9
[CHG] Device 00:18:E4:34:D1:C9 Connected: yes
[CHG] Device 00:18:E4:34:D1:C9 ServicesResolved: yes
Failed to connect: org.bluez.Error.NotAvailable
[CHG] Device 00:18:E4:34:D1:C9 ServicesResolved: no
[CHG] Device 00:18:E4:34:D1:C9 Connected: no
[CHG] Device 00:18:E4:34:D1:C9 RSSI: -71
[CHG] Device EA:BB:3D:70:DF:5A RSSI: -83
[CHG] Device 00:18:E4:34:D1:C9 RSSI: -57
[bluetooth]# trust 00:18:E4:34:D1:C9
[CHG] Device 00:18:E4:34:D1:C9 Trusted: yes
Changing 00:18:E4:34:D1:C9 trust succeeded
```

```bash
$ sudo rfcomm bind 0 00:18:E4:34:D1:C9
$ echo "hi" > /dev/rfcomm0
```

