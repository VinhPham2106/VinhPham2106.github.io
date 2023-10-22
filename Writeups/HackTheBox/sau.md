# SAU
* Difficulty: Easy
* Tag: POC, SSRF, sudo -l

## About this Box:
Here's the main ideas behind this box: 
* Port 80 is not accesible because there's a proxy on 55555
* Technologies used are [Request-Baskets](https://github.com/darklynx/request-baskets) and [Maltrail v0.53](https://github.com/stamparm/maltrail/blob/master/README.md).
* SSRF vulnerability that leads to RCE, but POC is searchable
* sudo -l gives system.ctl trail.service, and [GTFOBin](https://gtfobins.github.io/) does the rest.

## Conclusion:
* I hate ones where POC is available and enumaration is easy
* Ready for normal difficulty boxes now.
