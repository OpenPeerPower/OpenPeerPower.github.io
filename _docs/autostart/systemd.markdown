---
title: "Autostart using systemd"
description: "Instructions on how to setup Open Peer Power to launch on boot using systemd."
redirect_from: /getting-started/autostart-systemd/
---

Newer Linux distributions are trending towards using `systemd` for managing daemons. Typically, systems based on Fedora, ArchLinux, or Debian (8 or later) use `systemd`. This includes Ubuntu releases including and after 15.04, CentOS, and Red Hat. If you are unsure if your system is using `systemd`, you may check with the following command:

```bash
$ ps -p 1 -o comm=
```

If the preceding command returns the string `systemd`, continue with the instructions below.

A service file is needed to control Open Peer Power with `systemd`. The template below should be created using a text editor. Note, root permissions via `sudo` will likely be needed. The following should be noted to modify the template:

- `ExecStart` contains the path to `opp` and this may vary. Check with `whereis opp` for the location.
- For most systems, the file is `/etc/systemd/system/open-peer-power@YOUR_USER.service` with YOUR_USER replaced by the user account that Open Peer Power will run as (normally `openpeerpower`).  In particular, this is the case for Ubuntu 16.04.
- If unfamiliar with command-line text editors, `sudo nano -w [filename]` can be used with `[filename]` replaced with the full path to the file.  Ex. `sudo nano -w /etc/systemd/system/open-peer-power@YOUR_USER.service`.  After text entered, press CTRL-X then press Y to save and exit.
- If you're running Open Peer Power in a Python virtual environment or a Docker container, please skip to the appropriate template listed below.

```text
[Unit]
Description= Open Peer Power
After=network-online.target

[Service]
Type=simple
User=%i
ExecStart=/usr/bin/opp

[Install]
WantedBy=multi-user.target
```

### Python virtual environment

If you've setup Open Peer Power in `virtualenv` following our [Python installation guide](/getting-started/installation-virtualenv/) or [manual installation guide for Raspberry Pi](/getting-started/installation-raspberry-pi/), the following template should work for you. If Open Peer Power install is not located at `/srv/openpeerpower`, please modify the `ExecStart=` line appropriately. `YOUR_USER` should be replaced by the user account that Open Peer Power will run as (e.g `openpeerpower`).

```text
[Unit]
Description= Open Peer Power
After=network-online.target

[Service]
Type=simple
User=%i
ExecStart=/srv/openpeerpower/bin/opp -c "/home/%i/.openpeerpower"

[Install]
WantedBy=multi-user.target
```

### Docker

If you want to use Docker, the following template should work for you.

```text
[Unit]
Description= Open Peer Power
Requires=docker.service
After=docker.service

[Service]
Restart=always
RestartSec=3
ExecStart=/usr/bin/docker run --name=open-peer-power-%i -v /home/%i/.openpeerpower/:/config -v /etc/localtime:/etc/localtime:ro --net=host openpeerpower/open-peer-power
ExecStop=/usr/bin/docker stop -t 2 open-peer-power-%i
ExecStopPost=/usr/bin/docker rm -f open-peer-power-%i

[Install]
WantedBy=multi-user.target
```

### Next Steps

You need to reload `systemd` to make the daemon aware of the new configuration.

```bash
sudo systemctl --system daemon-reload
```

To have Open Peer Power start automatically at boot, enable the service.

```bash
sudo systemctl enable open-peer-power@YOUR_USER
```

To disable the automatic start, use this command.

```bash
sudo systemctl disable open-peer-power@YOUR_USER
```

To start Open Peer Power now, use this command.
```bash
sudo systemctl start open-peer-power@YOUR_USER
```

You can also substitute the `start` above with `stop` to stop Open Peer Power, `restart` to restart Open Peer Power, and 'status' to see a brief status report as seen below.

```bash
$ sudo systemctl status open-peer-power@YOUR_USER
● open-peer-power@fab.service - Open Peer Power for YOUR_USER
   Loaded: loaded (/etc/systemd/system/open-peer-power@YOUR_USER.service; enabled; vendor preset: disabled)
   Active: active (running) since Sat 2016-03-26 12:26:06 CET; 13min ago
 Main PID: 30422 (opp)
   CGroup: /system.slice/system-home\x2dassistant.slice/open-peer-power@YOUR_USER.service
           ├─30422 /usr/bin/python3 /usr/bin/opp
           └─30426 /usr/bin/python3 /usr/bin/opp
[...]
```

To get Open Peer Power's logging output, simple use `journalctl`.

```bash
sudo journalctl -f -u open-peer-power@YOUR_USER
```

Because the log can scroll quite quickly, you can select to view only the error lines:
```bash
sudo journalctl -f -u open-peer-power@YOUR_USER | grep -i 'error'
```

When working on Open Peer Power, you can easily restart the system and then watch the log output by combining the above commands using `&&`

```bash
sudo systemctl restart open-peer-power@YOUR_USER && sudo journalctl -f -u open-peer-power@YOUR_USER
```

### Automatically restarting Open Peer Power on failure

If you want to restart the Open Peer Power service automatically after a crash, add the following lines to the `[Service]` section of your unit file:

```text
Restart=on-failure
RestartSec=5s
```
