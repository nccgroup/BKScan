
# BKScan

BlueKeep (CVE-2019-0708) scanner that works both unauthenticated and
authenticated (i.e. when Network Level Authentication ([NLA](https://blogs.technet.microsoft.com/msrc/2019/05/14/prevent-a-worm-by-updating-remote-desktop-services-cve-2019-0708/)) is enabled).

Requirements:

* A Windows RDP server
* If NLA is enabled on the RDP server, a valid user/password that is part of the "Remote Desktop Users" group

It is based on [FreeRDP](https://github.com/FreeRDP/FreeRDP) and uses [Docker](https://www.docker.com/)
to ease compilation/execution. It should work on any UNIX environment and has been tested mainly on
Linux/Ubuntu.

# Usage

## Building

Install pre-requisites:

```
sudo apt-get install docker.io
```

Build the custom FreeRDP client inside the Docker container named `bkscan`:

```
$ git clone https://github.com/nccgroup/BKScan.git
$ cd BKScan
$ sudo docker build -t bkscan .
[...]
Successfully built f7666aeb3259
Successfully tagged bkscan:latest
```

## Running

Invoke the `bkscan.sh` script from your machine. It will invoke the custom FreeRDP client inside
the newly created `bkscan` Docker container:

```
$ sudo ./bkscan.sh -h
Usage:
./bkscan.sh -t <target_ip> [-P <target_port>] [-u <user>] [-p <password>] [--debug]
```

### Target with NLA enabled and valid credentials

Against a vulnerable Windows 7 with NLA enabled and valid credentials.

```
$ sudo ./bkscan.sh -t 192.168.119.141 -u user -p password
[+] Targeting 192.168.119.141:3389...
[+] Using provided credentials, will support NLA
[-] Max sends reached, please wait to be sure...
[!] Target is VULNERABLE!!!
```

Against a Windows 10 (non-vulnerable) or patched Windows 7 with NLA enabled and valid credentials:

```
$ sudo ./bkscan.sh -t 192.168.119.133 -u user -p password
[+] Targeting 192.168.119.133:3389...
[+] Using provided credentials, will support NLA
[-] Max sends reached, please wait to be sure...
[*] Target appears patched.
```

### Target with NLA enabled and non-valid credentials

Against a Windows 7 (vulnerable or patched) which NLA enabled
but that we are scanning with a client without NLA support:

```
$ sudo ./bkscan.sh -t 192.168.119.141
[+] Targeting 192.168.119.141:3389...
[+] No credential provided, won't support NLA
[-] Connection reset by peer, NLA likely to be enabled. Detection failed.
```

Against a Windows 7 (vulnerable or patched) with NLA enabled and valid credentials
but user is not part of the "Remote Desktop Users" group:

```
$ sudo ./bkscan.sh -t 192.168.119.141 -u test -p password
[+] Targeting 192.168.119.141:3389...
[+] Using provided credentials, will support NLA
[-] NLA enabled, credentials are valid but user has insufficient privileges. Detection failed.
```

Against a Windows 7 (vulnerable or patched) with NLA enabled and non-valid credentials:

```
$ sudo ./bkscan.sh -t 192.168.119.141 -u user -p badpassword
[+] Targeting 192.168.119.141:3389...
[+] Using provided credentials, will support NLA
[-] NLA enabled and access denied. Detection failed.
```

Against a Windows 10 (non-vulnerable) with NLA enabled and non-valid credentials:

```
$ sudo ./bkscan.sh -t 192.168.119.133 -u user -p badpassword
[+] Targeting 192.168.119.133:3389...
[+] Using provided credentials, will support NLA
[-] NLA enabled and logon failure. Detection failed.
```

Note: the difference in output between Windows 7 and Windows 10 is likely due to the Windows CredSSP versions
and your output may differ.

### Target with NLA disabled

Against a vulnerable Windows XP (no NLA support):

```
$ sudo ./bkscan.sh -t 192.168.119.137
[+] Targeting 192.168.119.137:3389...
[+] No credential provided, won't support NLA
[-] Max sends reached, please wait to be sure...
[!] Target is VULNERABLE!!!
```

### Target without RDP disabled

Against a Windows 7 with RDP disabled or blocked port:

```
$ sudo ./bkscan.sh -t 192.168.119.142
[+] Targeting 192.168.119.142:3389...
[+] No credential provided, won't support NLA
[-] Can't connect properly, check IP address and port.
```

# Thanks

Special thanks to @JaGoTu and @zerosum0x0 for releasing their Unauthenticated CVE-2019-0708 "BlueKeep"
Scanner, see [here](https://github.com/zerosum0x0/CVE-2019-0708). The BKScan scanner in this repo works
similarly to their 
[scanner](https://zerosum0x0.blogspot.com/2019/05/avoiding-dos-how-bluekeep-scanners-work.html) but has 
been ported to FreeRDP to support NLA.

Thank you to mi2428 for releasing a script to run FreeRDP in Docker, see [here](https://github.com/mi2428/docker-xfreerdp).

Also thank you to the following people for contributing:

* [nikallass](https://twitter.com/is_n3ws)

# Problems?

If you have a problem with the BlueKeep scanner, please create an issue on this github repository
with the detailed output using `./bkscan.sh --debug`.

## Known issues

### Failed to open display

Some recent versions of Linux (e.g. Ubuntu 18.04 or Kali 2019.2 Rolling) do not play well with the
`$DISPLAY` and `$XAUTHORITY` environment variables. 

```
$ sudo ./bkscan.sh -t 192.168.119.137
[+] Targeting 192.168.119.137:3389...
[+] No credential provided, won't support NLA
[07:58:35:866] [1:1] [ERROR][com.freerdp.client.x11] - failed to open display: :0
[07:58:35:866] [1:1] [ERROR][com.freerdp.client.x11] - Please check that the $DISPLAY environment variable is properly set.
```

It works fine on a fresh installation of Ubuntu 18.04 but not on an installation I have used for a
while so I am blaming some updated X11-related package or configuration.

[docker-org](https://github.com/rocker-org/rocker/wiki/Allowing-GUI-windows#linux-hosts)
documents this and proposes a solution but I haven't been able to have it working myself. So I am not sure
they are describing the same issue. If you have this issue initially and are able to fix it, please feel
free to do a PR.

# Contact

* [@saidelike](https://twitter.com/saidelike)