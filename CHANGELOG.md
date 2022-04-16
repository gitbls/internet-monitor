# Changelog

## V2.0

* Instances enable multiple internet-monitors to run concurrently. If you're using an action script of any sort, See README for details, or refer to sample-action.
  * Log information includes [instance-name], and default instance name is 'primary'
* Renamed some switches
  * `--internet` is now `--monitorip`
  * `--noping` is now `--ddnsonly`. Only track/report External IP Address. Do not check internet availability, etc.
  * `--ping` is now `--addping`, which takes a comma-separated list of additional ip addresses to monitor
* Added new switches
  * `--instance` arg defines the instance name. The default is "primary" if not specified. Only required to run multiple internet-monitors concurrently on the same system
  * `--noextip` Do not track/report External IP Address changes
* Defend against a broken *whats my IP* service returning a non-IPV4 address and try the next configured service
* Code cleanups
