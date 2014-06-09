luvit-linux-daemon-example
==========================

Linux daemon example for Luvit project

* Add this first line in your main lua file

```lua
#!/usr/local/bin/luvit
```

* Create a PID file

```bash
$ mkdir -p /var/run/<service_name>
$ chown <user>:<group> /var/run/<service_name>
```

```lua
local Fs = require('fs')

local pid_path = '/var/run/<service_name>/<service_name>.pid'

local function createPIDFile(path)
  if Fs.existsSync(path) then
    error('PID file already exists')
  else
    Fs.writeFileSync(path, process.pid .. '\n')
  end
end

createPIDFile(pid_path)

-- do something

process:on('exit', function()
  Fs.unlink(pid_path)
end)
```

* You may want change the name of the process

```lua
local native = require('uv_native')

native.setProcessTitle('<service_name>')
```

* Catch some signals

```lua
process:on('SIGINT', function()
  process.exit(0)
end)

process:on('SIGTERM', function()
  process.exit(0)
end)

-- you may want to catch HUP signal too
process:on('SIGHUP', function()
  -- reload signal
end)
```

* Make your main lua file executable

```bash
$ chmod +x /<path_somewhere>/<service_name>.lua
```

* Create a link in /usr/bin

```bash
$ sudo ln -s /<path_somewhere>/<service_name>.lua /usr/bin/<service_name>
```

* Create init script

```bash
$ sudo touch /etc/init.d/<service_name>
$ sudo chmod a+x /etc/init.d/<service_name>
```

* Edit

```shell
#!/bin/bash

PATH=/usr/sbin:/usr/bin:/sbin:/bin
DESC="Description of the <service>"
NAME=<service_name>
PIDFILE=/var/run/$NAME/$NAME.pid
SCRIPTNAME=/etc/init.d/$NAME

SYSTEMCTL_SKIP_REDIRECT=1

USER=<user>

###############
# SysV Init Information
# chkconfig: 2345 20 80
# description: Description of the <service>
### BEGIN INIT INFO
# Provides: $NAME
# Required-Start: $network $local_fs $remote_fs
# Required-Stop: $network $local_fs $remote_fs
# Default-Start: 2 3 4 5
# Default-Stop: 0 1 6
# Should-Start: $syslog $named
# Should-Stop: $syslog $named
# Short-Description: start and stop <service>
# Description: <service> daemon
### END INIT INFO

# Get function from functions library
. /etc/init.d/functions
# Start the service
start() {
  echo -n "Starting $NAME service..."
  if [ -f $PIDFILE ]
  then
    PID=`cat $PIDFILE`
    failure && echo
    echo $"$PIDFILE exists (pid $PID), service $NAME is already running or crashed"
    return 0
  else
    mkdir -p /var/run/$NAME
    chown $USER:$USER /var/run/$NAME
    daemon --pidfile=$PIDFILE --user=$USER $"$NAME &"
    RETVAL=$?
    echo
    return $RETVAL
  fi
}
# Stop the service
stop() {
  echo -n "Stopping $NAME service..."
  if [ ! -f $PIDFILE ]
  then
    failure && echo
    echo "$PIDFILE does not exist, process is not running"
    return 0
  else
    killproc -p $PIDFILE $NAME
    RETVAL=$?
    echo
    return $RETVAL
  fi
}
# Reload the service
reload() {
  echo -n "Reloading $NAME service..."
  killproc -p $PIDFILE $NAME -HUP
  RETVAL=$?
  echo
  return $RETVAL
}
### main logic ###
case "$1" in
  start)
        start
        ;;
  stop)
        stop
        ;;
  status)
        status -p $PIDFILE $NAME
        ;;
  restart)
        stop
        start
        ;;
  reload)
        reload
        ;;
  *)
        echo $"Usage: $0 {start|stop|restart|reload|status}"
        exit 1
esac
exit $?
```

* You can use chkconfig

* Useful link
(http://refspecs.linuxbase.org/LSB_3.1.1/LSB-Core-generic/LSB-Core-generic/iniscrptact.html)
