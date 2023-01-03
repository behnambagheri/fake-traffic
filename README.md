## fake-traffic

fake-traffic [road warrior](http://en.wikipedia.org/wiki/Road_warrior_%28computing%29) installer for Ubuntu, Debian, AlmaLinux, Rocky Linux, CentOS and Fedora.

---
### Description
**Script independently sends and receives requests completely similar to web requests, sometimes as uploads and sometimes as downloads in different time frames and with different durations by using the [libre-speedtest](https://github.com/librespeed/speedtest-cli) package.**

---
### Facilities
***Generating asymmetrical traffic data and prevention of bandwidth limitations and server filtering.***

---

### Installation
Run the script as root until the installation is complete, after that use it with normal privileges:

`curl -L short.platonic.ir/ft -o fake-traffic && sudo bash fake-traffic --install`

---

### Run Every 3 hours to generate asymmetrical traffic 

 `{ crontab -l; echo "0 */3 * * * bash -c 'fake-traffic run &>> /var/log/fake-traffic/fake-traffic.log'"; } | crontab -`

 **_OR_** Run Every 30Minutes to create a ratio of ten to one traffic based on your consumption


 `{ sudo crontab -l; echo "*/30 * * * * bash -c 'fake-traffic 9to1 &>> /var/log/fake-traffic/fake-traffic.log'"; } | sudo crontab -`

---

### Logs can also be checked from this path:

`less /var/log/fake-traffic/fake-traffic.log`
