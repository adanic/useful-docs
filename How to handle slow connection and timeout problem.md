# How to handle slow connection and timeout problem on CentOS7 with Nginx as webserver

### Try to solve it programmaticaly!
Before setting any of these configurations try to find the answer in you code and pray to find it there! That's the simpelest solution. If there is no answer for your questions in your logs you can try the following steps.
Good Luck!!!
## OS Tuning
Go to `/etc/sysctl.conf` and set the following configurations
```properties
fs.file-max=65536
# Disabling IPv6
net.ipv6.conf.all.disable_ipv6=1
net.ipv6.conf.default.disable_ipv6=1
# Recycle and reuse the connections in TIME_WAIT state
net.ipv4.tcp_tw_recycle=1
net.ipv4.tcp_tw_reuse=1
# Some useful configurations based on following link recommendations
# https://stackoverflow.com/questions/410616/increasing-the-maximum-number-of-tcp-ip-connections-in-linux
net.core.somaxconn=1024
net.core.netdev_max_backlog=2000
net.ipv4.tcp_max_syn_backlog=2048
```

Go to `/etc/security/limits.conf` and append following configuration at end of it
```
*         hard    nofile      65536
*         soft    nofile      65536
# Increase number of file and process limit for the following users
root      hard    nofile      65536
root      soft    nofile      65536
nginx     hard    nofile      65536
nginx     soft    nofile      65536
nginx     soft    nproc      256980
nginx     hard    nproc      256980
# Not recomended for projects other than kariz ;)
kariz     hard    nofile      65536
kariz     soft    nofile      65536
kariz     soft    nproc      256980
kariz     hard    nproc      256980
```
Reload system configuration using the following command
`sysctl -p`
Expected output is somthing like:
```
fs.file-max = 65536
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_tw_reuse = 1
net.core.somaxconn = 1024
net.core.netdev_max_backlog = 2000
net.ipv4.tcp_max_syn_backlog = 2048
```
### Increase `txquelen`
Default value is 1000. Check it using `ifconfig` then do the following(replace `$interface`)
`ifconfig $interface txqueuelen 5000`

## Nginx tuning
Set or change the following configurations in nginx conf:
```
worker_processes  auto;
worker_rlimit_nofile 300000;
events {
	    worker_connections  10000;
	}
```

## Some useful commands
To check limit  configuration for a user do the following (replace `$user`):

####Change user

``su - $user``

####Check soft limit

``ulimit -Sn``

####Check hard limit

``ulimit -Hn``

####Check number of open-files of a process (replace `$pid`):

``ls -1 /proc/$pid/fd | wc -l``

####Check maximum number of open-files (replace `$pid`):

``grep 'Max open files' /proc/$pid/limits | cut -d ' ' -f 15``

## Related links
- https://stackoverflow.com/questions/410616/increasing-the-maximum-number-of-tcp-ip-connections-in-linux
- https://wiki.geant.org/display/public/EK/InterfaceQueueLength
