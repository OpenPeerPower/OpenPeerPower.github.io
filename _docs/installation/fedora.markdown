---
title: "Installation on Fedora"
description: "Installation of Open Peer Power on your Fedora computer."
---

[Fedora](https://fedoraproject.org) is an operating system based on the Linux kernel, developed by the community-supported Fedora Project. There are releases for x86 and x86_64 including ARM and other architectures. 

Install the development package of Python.

```bash
sudo dnf -y install python3-devel redhat-rpm-config
```

To isolate the Open Peer Power installation a [`venv`](https://docs.python.org/3/library/venv.html) is handy. First create a new directory to store the installation and adjust the permissions.

```bash
sudo mkdir -p /opt/openpeerpower
sudo useradd -rm openpeerpower -G dialout
sudo chown -R openpeerpower:openpeerpower /opt/openpeerpower
```

Now switch to the new directory, setup the `venv`, and activate it.

```bash
sudo -u openpeerpower -H -s
cd /opt/openpeerpower
python3.8 -m venv .
source bin/activate
```

Install Open Peer Power itself.

```bash
pip3 install openpeerpower colorlog
```

Check the [autostart](/docs/autostart/systemd/) section in the documentation for further details and the [Firewall section](/docs/installation/troubleshooting/#no-access-to-the-frontend) if you want to access your Open Peer Power installation.
