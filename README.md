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
* `sudo nano /etc/systemd/system/internet-monitor.service` &mdash; Edit the ExecStart command line to adjust as needed.
* `sudo systemctl daemon-reload`
* `sudo systemctl enable [--now] internet-monitor` &mdash; Enable the service to start on the next boot. Use `--now` to start it now as well (good for testing!)

## Configuration

internet-monitor can be run from a terminal with logging to stdout. It is best run as a service with logging done to the system log. The service file internet-monitor.service is configured to write to the system log.

internet-monitor takes several switches:

* `--interval seconds` &mdash; Sets the number of seconds between internet IP ping checks. The default is 10 seconds.
* `--internet IP` &mdash; Sets the internet IP address to ping. The default is 1.1.1.1
* `--ping IP` &mdash; Sets an additional list of IP addresses to ping. The default is "". See Usage Hints below.
* `--pingwait seconds` &mdash; Sets the number of seconds to wait for a response to a ping. The default is 1 second.
* `--datefmt str` &mdash; Sets the date format string for output to stdout. The default is "+%Y-%m-%d %H:%M:%S". The plus sign is required. This is ignored if logging to syslog.
* `--action /full/path/to/script` &mdash; Provides an action script to be called on internet connectivity state transitions. See Action Script below.
* `--syslog` &mdash; By default internet-monitor writes to stdout. `--syslog` redirects the output to the system log.
* `--version` &mdash; Prints the internet-monitor version information and exits.

## Action Script

If `--action /full/path/to/script` is provided on the command line, internet-monitor will call the script in the following instances. In each case, the syslog argument will be 0 to write to syslog, 1 to write to stdout.

* **On startup** &mdash; Arguments provided are: "start" "date/time (canonical format)" syslog. 
* **Internet goes offline** &mdash; Arguments are: "offline" "date/time (canonical format)" syslog. 
* **Internet goes online** &mdash; Arguments are: "online" "date/time (canonical format)" syslog offline-duration. offline-duration is in the format "days:hours:minutes:seconds"

See the sample action script on this github for further information

## Usage Hints

In the typical use case, you want to know if your connection to the internet has been interrupted. You can check for that by using an Internet IP address such as 1.1.1.1, 8.8.8.8, etc. These are large internet hosts run by commercial organizations and have the plenty of network capacity. If you have your own Internet IP address outside of your LAN (e.g., on a VPC instance in the cloud), you can of course use that if it responds to ping requests.

The Internet can also become unavailable if your router has crashed. You can use the `--ping` switch to have internet-monitor also check your router, and report its availability as well to help with early diagnosis. If internet-monitor reports that your internet is down and your router is down as well, you'll quickly know that the problem is with your router rather than your Internet connection or your ISP.

Some home internet configurations have two routers: one for the ISP, and a personal router sitting behind it. In this case, internet-monitor can be used to check both routers by using `--ping myrouter-ip,isp-router-ip`. For example, `--ping 192.168.16.1,192.168.1.1`. Your IP addresses will most likely be different from these. The order of the IP addresses is your preference.

You can use the command `sudo journalctl | grep Internet-Monitor` to quickly scan the system log for Internet outages.



