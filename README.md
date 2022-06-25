# Description

The system is designed to automatically switch (reassign the default gateway) to a lower priority ISP if several providers are configured on the Mikrotik router, and one or more of higher priority ISP fail.

# Requirements

Mikrotik router must be configured with several ISPs, accordingly, several default gateways.

# How it works

- Every minute scheduler runs script "chk"
- Script "chk" checks the availability of each ISP channel using the external public always available IP addresses (in our case DNS. 77.88.8.1 - Yandex, 208.67.222.222 - OpenDNS), using specially marked routes on the Mikrotik router
- If higher priority ISP are unavailable, a lower priority ISP is switched on (by switching the default gateway)

# Mikrotik router configuration

- On each physical interface binded to specific ISP add comments: **-wan1-**, **-wan2-**, etc.:

```
/interface ethernet
set comment="-wan1-" name=ether2-wan1
set comment="-wan2-" name=ether3-wan2
```

- On each route that acts as the default gateway of each provider, add comments: **-gw1-**, **-gw2-**, etc.:

```
/ip route
add comment="-gw1-" distance=1 dst-address=0.0.0.0/0 gateway=$ISP1_GW
add comment="-gw2-" distance=1 dst-address=0.0.0.0/0 gateway=$ISP2_GW
```

- Let's add routes with certain route-marks. The monitoring script will use these routes to monitor a specific channel.

It uses access through certain gateways to certain public DNS addresses in the global network. This is necessary to determine their availability through a specific provider and, accordingly, the unambiguous health of the provider.

```
/ip route
add comment="-chk_isp1_pingServer1-" distance=1 dst-address=77.88.8.1 gateway=[/ip route get value-name=gateway [find where comment~"-gw1-"]] routing-mark=chk_isp1_pingServer1
add comment="-chk_isp1_pingServer2-" distance=1 dst-address=208.67.222.222 gateway=[/ip route get value-name=gateway [find where comment~"-gw1-"]] routing-mark=chk_isp1_pingServer2
add comment="-chk_isp2_pingServer1-" distance=1 dst-address=77.88.8.1 gateway=[/ip route get value-name=gateway [find where comment~"-gw2-"]] routing-mark=chk_isp2_pingServer1
add comment="-chk_isp2_pingServer2-" distance=1 dst-address=208.67.222.222 gateway=[/ip route get value-name=gateway [find where comment~"-gw2-"]] routing-mark=chk_isp2_pingServer2
```

- Add script "chk" (corresponding to the quantity of providers on router)

```
/system script add name=chk
/system script edit chk source
```

Insert contents of "chk_isp2.script" OR "chk_isp3.script" to script "chk"

- Add scheduler task

```
/system scheduler add interval=1m name=chk on-event=chk policy=ftp,reboot,read,write,policy,test,password,sniff,sensitive,romon start-time=startup</code>
```
