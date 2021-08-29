# BIND Example usage

(TODO) - this document details quickly setting up BIND9 to use with `kube-vip-dns`.

## Deploy `BIND`

### Start the first time to generate configuration

```
export EXT_IP=192.168.0.1
docker run --name bind -d \
--restart=always   \
--publish $EXT_IP:53:53/tcp \
--publish $EXT_IP:53:53/udp \
--publish 10000:10000/tcp   \
--volume /srv/docker/bind:/data   \
sameersbn/bind:9.16.1-20200524
```

### Configure our first zone

`/srv/docker/bind/bind/etc/named.conf.local`

```
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

zone "kube-vip.local" IN {
    type master;
    file "/var/lib/bind/kube-vip.local.zone";
    allow-update { any; };
    notify no;
};
```

`/srv/docker/bind/bind/lib/kube-vip.local.zone`

```
; BIND db file for kube-vip.local

$TTL 86400

@       IN      SOA     ns1.kube-vip.local.      dan.kube-vip.io. (
                        2021082901	; serial number YYMMDDNN
                        28800           ; Refresh
                        7200            ; Retry
                        864000          ; Expire
                        86400           ; Min TTL
			)

                NS      ns1.kube-vip.local.
ns1.kube-vip.local. IN A 127.0.0.1
```


## Dynamic DNS

```
$ nsupdate
> server 192.168.0.22
> update add dan.kube-vip.local. 2 A 1.2.3.4
>
>
> ^C

$ dig @192.168.0.22 dan.kube-vip.local

; <<>> DiG 9.11.3-1ubuntu1.15-Ubuntu <<>> @192.168.0.22 dan.kube-vip.local
; (1 server found)
;; global options: +cmd
;; Got answer:
;; WARNING: .local is reserved for Multicast DNS
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 60506
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 22e2d5f87553535601000000612bcbb5d2999069d8aca6db (good)
;; QUESTION SECTION:
;dan.kube-vip.local.		IN	A

;; ANSWER SECTION:
dan.kube-vip.local.	2	IN	A	1.2.3.4

;; Query time: 0 msec
;; SERVER: 192.168.0.22#53(192.168.0.22)
;; WHEN: Sun Aug 29 19:02:29 BST 2021
;; MSG SIZE  rcvd: 91

dan@code:/srv/docker/bind/bind/lib$
```
