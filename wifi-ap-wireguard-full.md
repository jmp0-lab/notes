# RPI + OpenBSD = Wifi AP + Wireguard full

```
 Host_1 -- (bwfm0)[ Wifi AP + Wireguard client ](bse0)(wg0) == { WAN } == (wg0)(vio0)[ Wireguard server ] == { WAN }
```

Whole Wifi AP clients (192.168.44.0/24) traffic will be tunneled with Wireguard (192.168.10.0/24).
X.X.X.X is a public address of Wireguard Server (YYY is a public port).

Generate keys (key{1,2,3,4}) according to the Wireguard documentation.

## Server

### /etc/hostname.wg0
```
wgkey [key1]
wgpeer [key2] wgaip 192.168.10.0/24

inet 192.168.10.1/24
wgport YYY

up
```
### /etc/pf.conf
```
pass in on wg0
pass in on egress proto udp from any to any port YYY
pass out quick on egress from (wg0:network) to any nat-to (egress:0)
```

## Client
### /etc/hostname.bwfm0
```
inet 192.168.44.1 255.255.255.0 NONE media autoselect mode 11n mediaopt hostap nwid TestName wpakey TestPassword
up
```
### /etc/hostname.wg0
```
wgkey [key3]
wgpeer [key4] wgendpoint X.X.X.X YYY wgaip 0.0.0.0/0 wgpka 20

inet 192.168.10.2/24
up

!route del default
!route add X.X.X.X/32 192.168.1.1
!route add default 192.168.10.1
```

### /etc/pf.conf
```
pass out quick on wg0 from bwfm0:network to any nat-to (wg0)
block out quick on bse0 from any to !X.X.X.X
```
