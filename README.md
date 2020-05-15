# ec2-install-supervisord

This document to help you install service Supervisord for web developer on EC2 (Extractly Amazon Linux Ami 2018.03).

## 1. Install supervisord 
Here are the first setup commands
```
$ sudo pip install supervisor
$ echo_supervisord_conf 
$ sudo su - 
$ echo_supervisord_conf > /etc/supervisord.conf 
```
Create the init scripts 
```
$ sudo nano /etc/init.d/supervisord
```
Copy below scripts and paste into init scripts previous step
```
#!/bin/bash
#
# supervisord   Startup script for the Supervisor process control system
#
# Author:       Mike McGrath <mmcgrath@redhat.com> (based off yumupdatesd)
#               Jason Koppe <jkoppe@indeed.com> adjusted to read sysconfig,
#                   use supervisord tools to start/stop, conditionally wait
#                   for child processes to shutdown, and startup later
#               Erwan Queffelec <erwan.queffelec@gmail.com>
#                   make script LSB-compliant
#
# chkconfig:    345 83 04
# description: Supervisor is a client/server system that allows \
#   its users to monitor and control a number of processes on \
#   UNIX-like operating systems.
# processname: supervisord
# config: /etc/supervisord.conf
# config: /etc/sysconfig/supervisord
# pidfile: /var/run/supervisord.pid
#
### BEGIN INIT INFO
# Provides: supervisord
# Required-Start: $all
# Required-Stop: $all
# Short-Description: start and stop Supervisor process control system
# Description: Supervisor is a client/server system that allows
#   its users to monitor and control a number of processes on
#   UNIX-like operating systems.
### END INIT INFO

# Source function library
. /etc/rc.d/init.d/functions

# Source system settings
if [ -f /etc/sysconfig/supervisord ]; then
    . /etc/sysconfig/supervisord
fi

# Path to the supervisorctl script, server binary,
# and short-form for messages.
supervisorctl=/usr/local/bin/supervisorctl
supervisord=${SUPERVISORD-/usr/local/bin/supervisord}
prog=supervisord
pidfile=${PIDFILE-/tmp/supervisord.pid}
lockfile=${LOCKFILE-/var/lock/subsys/supervisord}
STOP_TIMEOUT=${STOP_TIMEOUT-60}
OPTIONS="${OPTIONS--c /etc/supervisord.conf}"
RETVAL=0

start() {
    echo -n $"Starting $prog: "
    daemon --pidfile=${pidfile} $supervisord $OPTIONS
    RETVAL=$?
    echo
    if [ $RETVAL -eq 0 ]; then
        touch ${lockfile}
        $supervisorctl $OPTIONS status
    fi
    return $RETVAL
}

stop() {
    echo -n $"Stopping $prog: "
    killproc -p ${pidfile} -d ${STOP_TIMEOUT} $supervisord
    RETVAL=$?
    echo
    [ $RETVAL -eq 0 ] && rm -rf ${lockfile} ${pidfile}
}

reload() {
    echo -n $"Reloading $prog: "
    LSB=1 killproc -p $pidfile $supervisord -HUP
    RETVAL=$?
    echo
    if [ $RETVAL -eq 7 ]; then
        failure $"$prog reload"
    else
        $supervisorctl $OPTIONS status
    fi
}

restart() {
    stop
    start
}

case "$1" in
    start)
        start
        ;;
    stop)
        stop
        ;;
    status)
        status -p ${pidfile} $supervisord
        RETVAL=$?
        [ $RETVAL -eq 0 ] && $supervisorctl $OPTIONS status
        ;;
    restart)
        restart
        ;;
    condrestart|try-restart)
        if status -p ${pidfile} $supervisord >&/dev/null; then
          stop
          start
        fi
        ;;
    force-reload|reload)
        reload
        ;;
    *)
        echo $"Usage: $prog {start|stop|restart|condrestart|try-restart|force-reload|reload}"
        RETVAL=2
    esac

    exit $RETVAL
```
Set scripts executable for all users
```
chmod a+x /etc/init.d/supervisord
```
Add supervisord as a service for chkconfig 
```
$ sudo chkconfig --add supervisord
```
Setup supervisord turn on with system boot **(*)**
```
$ sudo chkconfig supervisord on
```
## 2. Config execute supervisord
To add one or multiple service run control by supervisord, to do below works
```
$ sudo nano /etc/supervisord.conf
```
Go to EOF and folder(s) contain service config file(s) you want to run it by that command
```
[include]
files = /etc/supervisord.d/*.conf ;everywhere you want, here I use folder /etc/supervisord.d
```
Make sure your folder exsist
```
$ sudo mkdir /etc/supervisord.d
```
## 3. Supervisord command
Common commands to control that service
```
$ sudo service supervisord start
$ sudo service supervisord stop
$ sudo service supervisord restart
$ sudo service supervisord status
```
## 4. Example 
Example one service control running by supervisord 
```
$ sudo nano \etc\supervisord.d\redis-server.conf
```
File content:
```
[program:supervisord_redis_server]
process_name=%(program_name)s_%(process_num)02d
command=/usr/local/bin/redis-server
autostart=true
autorestart=true
user=root
redirect_stderr=true
stdout_logfile=/var/www/logs/redis.log
```

## Issues 
Sometime **(*)** setup finished but supervisord cannot turn on with system boot because your services can depend enviroment. 
Just start Supervisord with command and find out that logs for issues.
```
$ sudo service supervisord start
```

