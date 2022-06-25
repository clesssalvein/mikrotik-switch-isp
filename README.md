# Description

The system is designed to automatically switch (reassign the default gateway) to a lower priority ISP if several providers are configured on the Mikrotik router, and one or more of more priority ISP fail.

# Requirements

Mikrotik router must be configured with several ISPs, accordingly, several default gateways.

# How it works

- Every minute scheduler runs script "chk"
- Script "chk" checks the availability of each ISP channel using the external public always available IP addresses (in our case DNS), using specially marked routes on the Mikrotik router
- If higher priority channels are unavailable, a lower priority channel is switched on (by switching the default gateway)

# Mikrotik router configuration

On every interface binded to specific ISP add comments: **-wan1-**, **-wan2-**, etc.

<code>
/interface ethernet
set comment="-wan1-" name=ether2-wan1
set comment="-wan2-" name=ether3-wan2
</code>
  
На каждом маршруте, выполняющем роль шлюза по-умолчанию каждого провайдера, пропишем комментарии: -gw1-, -gw2- и т.д.:

/ip route

add comment="-gw1-" distance=1 dst-address=0.0.0.0/0 gateway=$ISP1_GW

add comment="-gw2-" distance=1 dst-address=0.0.0.0/0 gateway=$ISP2_GW

Добавим маршруты с определенными route-mark. Скрипт мониторинга будет использовать эти маршруты для мониторинга определенного канала:

    Здесь используется доступ через определенные шлюзы до определенных публичных DNS адресов в глобальной сети. Это необходимо для определения их доступности через определенного провайдера и, соответственно, однозначной работоспособности провайдера.

/ip route

add comment="-chk_isp1_pingServer1-" distance=1 dst-address=77.88.8.1 gateway=[/ip route get value-name=gateway [find where comment~"-gw1-"]] routing-mark=chk_isp1_pingServer1

add comment="-chk_isp1_pingServer2-" distance=1 dst-address=208.67.222.222 gateway=[/ip route get value-name=gateway [find where comment~"-gw1-"]] routing-mark=chk_isp1_pingServer2

add comment="-chk_isp2_pingServer1-" distance=1 dst-address=77.88.8.1 gateway=[/ip route get value-name=gateway [find where comment~"-gw2-"]] routing-mark=chk_isp2_pingServer1

add comment="-chk_isp2_pingServer2-" distance=1 dst-address=208.67.222.222 gateway=[/ip route get value-name=gateway [find where comment~"-gw2-"]] routing-mark=chk_isp2_pingServer2 
