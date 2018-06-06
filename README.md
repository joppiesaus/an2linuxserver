# AN2Linux - server
Sync Android notifications encrypted using TLS to a Linux desktop over WiFi, Mobile data or Bluetooth.

This is the server part of AN2Linux.

[Link to client part](https://github.com/rootkiwi/an2linuxclient/)

## About
AN2Linux is my first (not tiny) program / app and I've been working on it for
quite some time now.
I wanted to make an2linux because it was something I wanted and also to learn.

It's been a fun ride, I have learned a lot.

## Dependencies
I'm using archlinux but I've added here what I think to be the package
names for debian/ubuntu as well.

* **python (3.4+)**
```
Arch: python
Debian / Ubuntu: python3
```

* **python-gobject**
```
Arch: python-gobject
Debian / Ubuntu: python3-gi
```

* **openssl (1.0.1+)**

## Dependencies for bluetooth
* **python-pip**
```
Arch: python-pip
Debian / Ubuntu: python3-pip
```

* **BlueZ dev files**
```
Arch: bluez-libs
Debian / Ubuntu: libbluetooth-dev
```

* **PyBluez**
```
pip3 install pybluez
```

## How to use
First time just run `an2linuxserver.py` and follow the instructions.

AN2Linux uses TLS for encryption with both client and server authentication.
That means that the client (android) and the server (your computer)
need to exchange certificates first.

To initiate pairing you must have `an2linuxserver.py` running and then in the
android app press initiate pairing when in the process of adding a new server.

### For bluetooth
First you need to pair your phone and computer with each other (the normal
bluetooth pairing), and then do the certificate exchange as explained above.

Why use TLS over bluetooth when bluetooth is already encrypted?

Well, firstly, because I wanted to try :)

And secondly, from the little I've read about bluetooth it seems that the
encryption key size negotiated can be very small.

#### PyBluez not working?
I'm using archlinux so I'm going to tell you how it works for me, I have no
idea how it works on other distros. It may work out of the box or it may not, just try it.

This is the error I get without the fixes below:<br>
`bluetooth.btcommon.BluetoothError: (13, 'Permission denied')`

So, if you're not using archlinux but still are using systemd and get an error like
that maybe you should try something similar.

#### Edit bluetooth.service in an override file
```
systemctl edit bluetooth.service
```

#### Add the following lines
```
[Service]
ExecStart=
ExecStart=/usr/lib/bluetooth/bluetoothd -C
ExecStartPost=/bin/chmod 662 /var/run/sdp
```

#### then apply changes
```
systemctl daemon-reload
systemctl restart bluetooth.service
```

More info about this problem:
https://bbs.archlinux.org/viewtopic.php?id=201672.

## Config directory
First time when running `an2linuxserver.py` it will create the directory:
`$XDG_CONFIG_HOME/an2linux/`.

If `$XDG_CONFIG_HOME` is not set it defaults to: `$HOME/.config/an2linux/`.

In this config directory a few files will be created.

#### Config file
A default config file named `config` will be created with the settings:
- `tcp_server` **[on/off]** *default:* **on**
- `tcp_port` **[0-65535]** *default:* **46352**
- `bluetooth_server` **[on/off]** *default:* **off**
- `bluetooth_support_kitkat` **[on/off]** *default:* **off**
- `notification_timeout` **[integer]** *default:* **5**
- `list_size_duplicates` **[0-50]** *default:* **0**
- `ignore_duplicates_list_for_titles` **[comma-separated list]** *default:* **AN2Linux, title**

Open that config file for more detailed info about every setting.

#### Server RSA key and certificate
AN2Linux will generate a 4096 bit RSA key `rsakey.pem`  and a self-signed certificate `certificate.pem`.

Just delete those and restart an2linux if you want to generate a new key and certificate.

#### Trused certificates
After a successful certificate exchange an2linux will create a file named
`authorized_certs`.

Every line in that file will represent a trused certificate.

It will be in the format `SHA256:<certificate_fingerprint> <trused_certificate>`.

That first part `SHA256...` is not used by an2linux at all, it's just
added as a convenience to the user to help distinguish between multiple certificates.

#### Diffie–Hellman ephemeral
If the setting `bluetooth_support_kitkat` is turned `on` an2linux will also generate a file named `dhparam.pem`.

That is because [SSLEngine](https://developer.android.com/reference/javax/net/ssl/SSLEngine.html) does not support any
[ECDHE](https://en.wikipedia.org/wiki/Elliptic_curve_Diffie%E2%80%93Hellman) cipher suites
until from API 20+ (Android 5.0+).

## Run AN2Linux as a service
[Init/Service scripts](https://github.com/rootkiwi/an2linuxserver/tree/master/init)

## License
[GNU General Public License 3](https://www.gnu.org/licenses/gpl-3.0.html),
with the additional special 
exception to link portions of this program with the OpenSSL library.

See LICENSE for more details.
