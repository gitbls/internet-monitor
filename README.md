# internet-monitor
Monitors Internet connectivity

## Overview

internet-monitor regularly checks your internet connectivity and logs outages. It can additionally call an action script of your creation to do...whatever.

internet-monitor is easily installed and runs as a systemd service. 

## Installation

Download the internet-monitor files from GitHub, and edit the file /etc/systemd/system/internet-monitor.service as needed for your configuration.

* `sudo curl -L https://raw.githubusercontent.com/gitbls/internet-monitor/master/internet-monitor -o /usr/local/bin/internet-monitor`
* `sudo chmod 755 /usr/local/bin/internet-monitor`
* `sudo curl -L https://raw.githubusercontent.com/gitbls/internet-monitor/master/internet-monitor.service -o /etc/systemd/system/internet-monitor.service`
* `sudo curl -L https://raw.githubusercontent.com/gitbls/internet-monitor/master/internet-monitor@.service -o /etc/systemd/system/internet-monitor@.service`
* `sudo curl -L https://raw.githubusercontent.com/gitbls/internet-monitor/master/internet-monitor.instance -o /etc/default/internet-monitor.instance
* `sudo nano /etc/systemd/system/internet-monitor.service` &mdash; Edit the ExecStart command line to adjust as needed, *or* use instances. See below.
* `sudo systemctl daemon-reload`
* `sudo systemctl enable [--now] internet-monitor` &mdash; Enable the service to start on the next boot. Use `--now` to start it now as well (good for testing!)

## Configuration

internet-monitor can be run from a terminal with logging to stdout. It is best run as a service with logging done to the system log. The service file internet-monitor.service is configured to write to the system log.

internet-monitor takes several switches:

* `--action /full/path/to/script` &mdash; Provides an action script to be called on internet connectivity state transitions. See *Action Scripts* below.
* `--addping IP` &mdash; Sets an additional list of IP addresses to ping. The default is "". See Usage Hints below.
* `--datefmt str` &mdash; Sets the date format string for output to stdout. The default is "+%Y-%m-%d %H:%M:%S". The plus sign is required. This is ignored if logging to syslog.
* `--ddns&mdash; Check external IP address every --interval X --einterval seconds. If external IP address changes, call the action script 
* `--ddnsonly` &mdash; Only track/report External IP Address. Do not check internet availability, etc.
* `--einterval N` &mdash; Number of --interval waits between External IP Address check if --ddns specified. The default is 60 (10 minutes with `--interval 10`)
* `--interval seconds` &mdash; Sets the number of seconds between internet IP ping checks. The default is 10 seconds.
* `--monitorip IP` &mdash; Sets the internet IP address to monitor. The default is 1.1.1.1
* `--noextip`  &mdash; Do not monitor External IP Address.
* `--pingwait seconds` &mdash; Sets the number of seconds to wait for a response to a ping. The default is 1 second.
* `--pingcount N` &mdash; Specify that ping testing with a ping count of N should be done against the Internet IP Address.
* `--standby IP` &mdash; Secondary IP to monitor. Calls action script when it goes up or down
* `--syslog` &mdash; By default internet-monitor writes to stdout. `--syslog` redirects the output to the system log.
* `--version` &mdash; Prints the internet-monitor version information and exits.

## Action Scripts

If `--action /full/path/to/script` is provided on the command line, internet-monitor will call the script in the following instances. In each case, the syslog argument will be 0 to write to syslog, 1 to write to stdout.

* **On startup** &mdash; Arguments provided are: "start" "date/time (canonical format)" syslog. 
* **Internet goes offline** &mdash; Arguments are: "offline" "date/time (canonical format)" syslog. A return of "1" from the Action Script indicates that the internet is not really offline. A return of "0" or "" indicates that the internet is offline. See the Sample Action script for details.
* **Internet goes online** &mdash; Arguments are: "online" "date/time (canonical format)" syslog offline-duration. offline-duration is in the format "days:hours:minutes:seconds"
* **Ping test result**  &mdash;  Arguments are: "ping" datetime pingmin pingavg pingmax pingmdev loss msec
* **Standby IP goes offline** &mdash; Arguments are: "standby-offline" "date/time (canonical format)" standby-IP
* **Standby IP goes online** &mdash; Arguments are: "standby-online" "date/time (canonical format)" standby-IP
* **External IP adddress changes** &mdash; Arguments are: "new-external-ip" "old-external-ip". This will only be called if `--ddns` is specified on the internet-monitor command line.

See the sample action scripts [sample-action](https://github.com/gitbls/internet-monitor/blob/master/sample-action) and [failover-monitor-action](https://github.com/gitbls/internet-monitor/blob/master/failover-monitor-action) on this github for further information.

## Instances

internet-monitor supports multiple, concurrently-running instances. If not specified, the default instance name is "primary". Instances modify configuration processing, thus allowing multiple instances of internet-monitor to run on the same system simultaneously.

To specify an instance, use `--instance name`. If this is not used, a default `--instance primary` is applied. The configuration details for the instance are specified in /etc/default/internet-monitor.instancename. Create this file by copying the file /etc/default/internet-monitor.instance to /etc/default/internet-monitor.instancename and making the desired configuration changes. The file identifies the default settings. To change a value, simply uncomment the line and set the new value.

Once you have the configuration set, you can enable and start the service:

`sudo systemctl enable [--now] internet-monitor@instancename`

The instance name can be anything you want, except for the word `primary`, which is the name of the default instance.

The primary instance can be started either by using the service `internet-monitor@primary.service`, or by using the service `internet-monitor.service`, which uses the internet-monitor default settings, *aka* `primary`. In either case, internet-monitor will read (if it exists) /etc/default/internet-monitor.primary for settings. An instance setting file is required for all instances, except for the primary instance, where it is optional.

All syslog messages from internet-monitor include the instance name.

## Network Failover Monitor

The failover-monitor-action script is a working example of using internet-monitor to ensure that one of two WAN networks is always online. This is useful, for instance, if you have a faster, or otherwise more preferred primary connection, and a slower, or less preferred secondary connection, although that's not the only use case for it.

failover-monitor-action can either prefer the primary network or just ensure that one network is online and routing traffic.

Briefly, to configure the failover monitor:

* Install internet-monitor per the installation and configuration guide above
* Copy failover-monitor-action to /usr/local/bin
* `sudo systemctl stop internet-monitor`
* Edit /etc/systemd/system/internet-monitor.service and add --standby ip:ad:dd:re:ss and add the --action switch with the full path to the failover-monitor-action script
* Download the sample internet-monitor.defaults from this github to /etc/default/internet-monitor, and edit it to reflect your configuration
* `sudo systemctl start internet-monitor`, check the logs, and correct any issues found
* `sudo systemctl enable internet-monitor` to enable it to start automatically when the system reboots


## Usage Hints

In the typical use case, you want to know if your connection to the internet has been interrupted. You can check for that by using an Internet IP address such as 1.1.1.1, 8.8.8.8, etc. These are large internet hosts run by commercial organizations and have the plenty of network capacity. If you have your own Internet IP address outside of your LAN (e.g., on a VPC instance in the cloud), you can of course use that if it responds to ping requests.

The Internet can also become unavailable if your router has crashed. You can use the `--addping` switch to have internet-monitor also check your router, and report its availability as well to help with early diagnosis. If internet-monitor reports that your internet is down and your router is down as well, you'll quickly know that the problem is with your router rather than your Internet connection or your ISP.

Some home internet configurations have two routers: one for the ISP, and a personal router sitting behind it. In this case, internet-monitor can be used to check both routers by using `--addping myrouter-ip,isp-router-ip`. For example, `--addping 192.168.16.1,192.168.1.1`. Your IP addresses will most likely be different from these. The order of the IP addresses is your preference.

You can use the command `sudo journalctl | grep Internet-Monitor` to quickly scan the system log for Internet outages.
