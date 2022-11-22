# fake-traffic
**This script stand alone sends and receives requests completely similar to web requests, sometimes as uploads and sometimes as downloads in different time frames and with different durations by using the [libre-speedtest](https://github.com/librespeed/speedtest-cli) package.**


### Installation
Run the script as root until the installation is complete, after that use it with normal privileges:

`wget short.platonic.ir/ft -O fake-traffic && bash fake-traffic`

### Run Every 3 hours

 `{ crontab -l; echo "0 */3 * * * fake-traffic &>> ~/.fake-traffic.log"; } | crontab -`
 
### Logs can also be checked from this path:

`cat ~/.fake-traffic.log`