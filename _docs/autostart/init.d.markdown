---
title: "Autostart using init.d"
description: "Documentation about setting up Open Peer Power as a daemon running under init.d."
redirect_from: /getting-started/autostart-init.d/
---

 Open Peer Power can run as a daemon within init.d with the script below.

## 1. Copy script

Copy either the daemon script or the Python environment script at the end of this page to `/etc/init.d/opp-daemon` depending on your installation.

After that, set the script to be executable:

```bash
sudo chmod +x /etc/init.d/opp-daemon
```

## 2. Select a user

Create or pick a user that the Open Peer Power daemon will run under. Update script to set `RUN_AS` to the username that should be used to execute Open Peer Power.

## 3. Change `opp` executable and other variables if required

Some installation environments may require a change in the Open Peer Power executable `opp`. Update script to set `OPP_OPPBIN` to the appropriate `opp` executable path. Please also check the other variables for the appropriate value. In general the defaults should work

## 4. Install this service

```bash
sudo service opp-daemon install
```

## 5. Create logrotate rule

This logrotate script at `/etc/logrotate.d/openpeerpower` will create an outage of a few seconds every week at night. If you do not want this add `--log-rotate-days 7` to the `FLAGS` variable in the init script.

File `/var/log/openpeerpower/open-peer-power.log`:

```text
{
        rotate 7
        daily
        missingok
        notifempty
        delaycompress
        compress
        postrotate
                invoke-rc.d opp-daemon restart > /dev/null
        endscript
}

```

### 6. Restart Machine

That's it. Restart your machine and Open Peer Power should start automatically.

If Open Peer Power does not start, check the log file output for errors at `/var/log/openpeerpower/open-peer-power.log`

### Extra: Running commands before Open Peer Power executes

If any commands need to run before executing Open Peer Power (like loading a virtual environment), put them in PRE_EXEC. This command must end with a semicolon.

### Daemon script

```bash
#!/bin/sh
### BEGIN INIT INFO
# Provides:          opp
# Required-Start:    $local_fs $network $named $time $syslog
# Required-Stop:     $local_fs $network $named $time $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Description:       Home\ Assistant
### END INIT INFO

# /etc/init.d Service Script for Open Peer Power
# Created with: https://gist.github.com/naholyr/4275302#file-new-service-sh
PRE_EXEC=""
# Typically /usr/bin/opp
OPP_BIN="opp"
RUN_AS="openpeerpower"
PID_DIR="/var/run/opp"
PID_FILE="$PID_DIR/opp.pid"
CONFIG_DIR="/var/opt/openpeerpower"
LOG_DIR="/var/log/openpeerpower"
LOG_FILE="$LOG_DIR/open-peer-power.log"
FLAGS="-v --config $CONFIG_DIR --pid-file $PID_FILE --log-file $LOG_FILE --daemon"


start() {
  create_piddir
  if [ -f $PID_FILE ] && kill -0 $(cat $PID_FILE) 2> /dev/null; then
    echo 'Service already running' >&2
    return 1
  fi
  echo -n 'Starting service… ' >&2
  local CMD="$PRE_EXEC $OPP_BIN $FLAGS"
  su -s /bin/bash -c "$CMD" $RUN_AS
  if [ $? -ne 0 ]; then
    echo "Failed" >&2
  else
    echo 'Done' >&2
  fi
}

stop() {
  if [ ! -f "$PID_FILE" ] || ! kill -0 $(cat "$PID_FILE") 2> /dev/null; then
    echo 'Service not running' >&2
    return 1
  fi
  echo -n 'Stopping service… ' >&2
  kill $(cat "$PID_FILE")
  while ps -p $(cat "$PID_FILE") > /dev/null 2>&1; do sleep 1;done;
  rm -f $PID_FILE
  echo 'Done' >&2
}

install() {
  echo "Installing Open Peer Power Daemon (opp-daemon)"
  update-rc.d opp-daemon defaults
  create_piddir
  mkdir -p $CONFIG_DIR
  chown $RUN_AS $CONFIG_DIR
  mkdir -p $LOG_DIR
  chown $RUN_AS $LOG_DIR
}

uninstall() {
  echo "Are you really sure you want to uninstall this service? The INIT script will"
  echo -n "also be deleted! That cannot be undone. [yes|No] "
  local SURE
  read SURE
  if [ "$SURE" = "yes" ]; then
    stop
    remove_piddir
    echo "Notice: The config directory has not been removed"
    echo $CONFIG_DIR
    echo "Notice: The log directory has not been removed"
    echo $LOG_DIR
    update-rc.d -f opp-daemon remove
    rm -fv "$0"
    echo " Open Peer Power Daemon has been removed. Open Peer Power is still installed."
  fi
}

create_piddir() {
  if [ ! -d "$PID_DIR" ]; then
    mkdir -p $PID_DIR
    chown $RUN_AS "$PID_DIR"
  fi
}

remove_piddir() {
  if [ -d "$PID_DIR" ]; then
    if [ -e "$PID_FILE" ]; then
      rm -fv "$PID_FILE"
    fi
    rmdir -v "$PID_DIR"
  fi
}

case "$1" in
  start)
    start
    ;;
  stop)
    stop
    ;;
  install)
    install
    ;;
  uninstall)
    uninstall
    ;;
  restart)
    stop
    start
    ;;
  *)
    echo "Usage: $0 {start|stop|restart|install|uninstall}"
esac
```

### Python virtual environment

```bash
#!/bin/sh
### BEGIN INIT INFO
# Provides:          opp
# Required-Start:    $local_fs $network $named $time $syslog
# Required-Stop:     $local_fs $network $named $time $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Description:       Home\ Assistant
### END INIT INFO

# /etc/init.d Service Script for Open Peer Power
# Created with: https://gist.github.com/naholyr/4275302#file-new-service-sh
PRE_EXEC="cd /srv/openpeerpower; python3 -m venv .; source bin/activate;"
# Typically /usr/bin/opp
OPP_BIN="opp"
RUN_AS="openpeerpower"
PID_DIR="/var/run/opp"
PID_FILE="$PID_DIR/opp.pid"
CONFIG_DIR="/home/$RUN_AS/.openpeerpower"
LOG_DIR="/var/log/openpeerpower"
LOG_FILE="$LOG_DIR/open-peer-power.log"
FLAGS="-v --config $CONFIG_DIR --pid-file $PID_FILE --log-file $LOG_FILE --daemon"

start() {
  create_piddir
  if [ -f $PID_FILE ] && kill -0 $(cat $PID_FILE) 2> /dev/null; then
    echo 'Service already running' >&2
    return 1
  fi
  echo -n 'Starting service… ' >&2
  local CMD="$PRE_EXEC $OPP_BIN $FLAGS"
  su -s /bin/bash -c "$CMD" $RUN_AS
  if [ $? -ne 0 ]; then
    echo "Failed" >&2
  else
    echo 'Done' >&2
  fi
}

stop() {
  if [ ! -f "$PID_FILE" ] || ! kill -0 $(cat "$PID_FILE") 2> /dev/null; then
    echo 'Service not running' >&2
    return 1
  fi
  echo -n 'Stopping service… ' >&2
  kill $(cat "$PID_FILE")
  while ps -p $(cat "$PID_FILE") > /dev/null 2>&1; do sleep 1;done;
  rm -f $PID_FILE
  echo 'Done' >&2
}

install() {
  echo "Installing Open Peer Power Daemon (opp-daemon)"
  update-rc.d opp-daemon defaults
  create_piddir
  mkdir -p $CONFIG_DIR
  chown $RUN_AS $CONFIG_DIR
  mkdir -p $LOG_DIR
  chown $RUN_AS $LOG_DIR
}

uninstall() {
  echo "Are you really sure you want to uninstall this service? The INIT script will"
  echo -n "also be deleted! That cannot be undone. [yes|No] "
  local SURE
  read SURE
  if [ "$SURE" = "yes" ]; then
    stop
    remove_piddir
    echo "Notice: The config directory has not been removed"
    echo $CONFIG_DIR
    echo "Notice: The log directory has not been removed"
    echo $LOG_DIR
    update-rc.d -f opp-daemon remove
    rm -fv "$0"
    echo " Open Peer Power Daemon has been removed. Open Peer Power is still installed."
  fi
}

create_piddir() {
  if [ ! -d "$PID_DIR" ]; then
    mkdir -p $PID_DIR
    chown $RUN_AS "$PID_DIR"
  fi
}

remove_piddir() {
  if [ -d "$PID_DIR" ]; then
    if [ -e "$PID_FILE" ]; then
      rm -fv "$PID_FILE"
    fi
    rmdir -v "$PID_DIR"
  fi
}

case "$1" in
  start)
    start
    ;;
  stop)
    stop
    ;;
  install)
    install
    ;;
  uninstall)
    uninstall
    ;;
  restart)
    stop
    start
    ;;
  *)
    echo "Usage: $0 {start|stop|restart|install|uninstall}"
esac
```
