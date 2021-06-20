### About this repository 

This repository shows gNOI demo with Arista.  
It includes examples using gNOIc and gRPCurl.  

### About gRPC

gRPC - Google Remote Procedure Call   

gRPC uses protobuf and HTTP/2 

### About gNOI 

gNOI - gRPC Network Operations Interface   

gNOI defines a set of gRPC-based microservices for executing operational commands on network devices.  

gNOI repository https://github.com/openconfig/gnoi  

As example, this gNOI proto file https://github.com/openconfig/gnoi/blob/master/system/system.proto defines the service `System` with the RPC `Traceroute` and `Ping`
- Ping executes the ping command on the target and streams back the results
- Traceroute executes the traceroute command on the target and streams back the results  
- As you can see in the proto file, the field VRF is not defined for these messages  
  
### gNOI support on EOS 

https://eos.arista.com/eos-4-24-2f/gnoi/  

Examples:  
https://eos.arista.com/eos-4-22-1f/gnoi-ping/   
https://eos.arista.com/eos-4-22-1f/gnoi-traceroute/  

### EOS 

EOS switch configuration:   
```
hostname DC1-L2LEAF2A

ip name-server vrf MGMT 8.8.8.8

interface Management1
   description oob_management
   vrf MGMT
   ip address 10.73.1.118/24

username arista secret 0 arista 

ip access-list GNMI
   10 permit tcp any any eq gnmi

management api gnmi
   transport grpc def
      vrf MGMT
      ip access-group GNMI
   provider eos-native

management api http-commands
   protocol http
   no shutdown
```
Before to use gNOI ping and traceroute, lets run these commands locally: 
```
$ ssh arista@10.73.1.118
Password: 
Last login: Thu Jun  3 12:06:25 2021 from 10.73.1.3
DC1-L2LEAF2A>en
DC1-L2LEAF2A#bash

Arista Networks EOS shell

[arista@DC1-L2LEAF2A ~]$ ping  172.31.255.0 -c 2
PING 172.31.255.0 (172.31.255.0) 56(84) bytes of data.
64 bytes from 172.31.255.0: icmp_seq=1 ttl=63 time=24.6 ms
64 bytes from 172.31.255.0: icmp_seq=2 ttl=63 time=18.8 ms

--- 172.31.255.0 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 18.861/21.738/24.616/2.881 ms
[arista@DC1-L2LEAF2A ~]$ 
[arista@DC1-L2LEAF2A ~]$ traceroute -A 172.31.255.0
traceroute to 172.31.255.0 (172.31.255.0), 30 hops max, 60 byte packets
 1  10.90.90.1 (10.90.90.1) [!!]  26.636 ms  29.420 ms  32.113 ms
 2  172.31.255.0 (172.31.255.0) [!!]  52.764 ms  53.881 ms  63.213 ms
[arista@DC1-L2LEAF2A ~]$ 
[arista@DC1-L2LEAF2A ~]$ exit
logout
DC1-L2LEAF2A#exit
Connection to 10.73.1.118 closed.
```
### gNOI demo with Arista using gNOIc 

#### About gNOIc 

gNOIc is a gNOI CLI client   
https://github.com/karimra/gnoic  
 
#### Install gNOIc 
```
bash -c "$(curl -sL https://get-gnoic.kmrd.dev)"
```
```
$ gnoic version
version : 0.0.5
 commit : 26c6248
   date : 2021-05-12T10:12:55Z
 gitURL : https://github.com/karimra/gnoic
   docs : https://gnoic.kmrd.dev
```
#### Use gNMIc 
```
$ gnoic -a 10.73.1.118:6030 -u arista -p arista --insecure  system ping --destination 172.31.255.0 --count 2 --do-not-resolve
WARN[0000] "10.73.1.118:6030" could not lookup hostname: lookup 118.1.73.10.in-addr.arpa. on 127.0.0.53:53: no such host 
source: "172.31.255.0"
time: 31200000
bytes: 64
sequence: 1
ttl: 63
source: "172.31.255.0"
time: 33900000
bytes: 64
sequence: 2
ttl: 63
source: "172.31.255.0"
time: 1001000000
sent: 2
received: 2
min_time: 31251000
avg_time: 32590000
max_time: 33930000
std_dev: 1351000
```
```
$ gnoic -a 10.73.1.118:6030 -u arista -p arista --insecure  system traceroute --destination 172.31.255.0 --do-not-resolve
WARN[0000] "10.73.1.118:6030" could not lookup hostname: lookup 118.1.73.10.in-addr.arpa. on 127.0.0.53:53: no such host 
destination_name: "172.31.255.0"
destination_address: "172.31.255.0"
hops: 30
packet_size: 60
hop: 1
address: "10.90.90.1"
rtt: 21440000
hop: 1
address: "10.90.90.1"
rtt: 23011000
hop: 1
address: "10.90.90.1"
rtt: 31135000
hop: 2
address: "172.31.255.0"
rtt: 62216000
hop: 2
address: "172.31.255.0"
rtt: 63213000
hop: 2
address: "172.31.255.0"
rtt: 71079000
```
```
$ gnoic -a 10.73.1.118:6030 -u arista -p arista --insecure cert can-generate-csr
WARN[0000] "10.73.1.118:6030" could not lookup hostname: lookup 118.1.73.10.in-addr.arpa. on 127.0.0.53:53: no such host 
INFO[0000] "10.73.1.118:6030" key-type=KT_RSA, cert-type=CT_X509, key-size=2048: can_generate: true 
+------------------+------------------+
|   Target Name    | Can Generate CSR |
+------------------+------------------+
| 10.73.1.118:6030 | true             |
+------------------+------------------+
```

### gNOI demo with Arista using gRPCurl 

#### About gRPCurl 

gRPCurl  is a command-line tool that lets you interact with gRPC servers.   

#### Install GO 

```
$ go version
go version go1.16.4 linux/amd64
```
```
$ go env | grep 'GOROOT\|GOPATH'
```
```
$ echo $HOME
$ echo $GOROOT
$ echo $GOPATH 
$ echo $PATH
$ export GOROOT=/usr/local/go
$ export GOPATH=$HOME/go
$ export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
```

#### Get gNOI repository

```
$ mkdir -p $GOPATH/src/github.com/openconfig
$ git clone https://github.com/openconfig/gnoi.git $GOPATH/src/github.com/openconfig/gnoi 
```
```
$ ls $GOPATH/src/github.com/openconfig
gnoi
```

#### Install gRPCurl

```
$ go get github.com/fullstorydev/grpcurl
```
```
$ ls $GOPATH/pkg/mod/github.com/fullstorydev/
grpcurl@v1.8.1
```
```
$ go install github.com/fullstorydev/grpcurl/cmd/grpcurl@latest
```
```
ls $GOPATH/bin/
grpcurl 
```

#### Use gRPCurl
```
$ grpcurl -H 'username: arista'  -H 'password: arista' -d '{"destination": "172.31.255.0", "count": 2, "do_not_resolve":true}' -import-path ${GOPATH}/src -proto github.com/openconfig/gnoi/system/system.proto -plaintext 10.73.1.118:6030 gnoi.system.System/Ping
{
  "source": "172.31.255.0",
  "time": "29800000",
  "bytes": 64,
  "sequence": 1,
  "ttl": 63
}
{
  "source": "172.31.255.0",
  "time": "25200000",
  "bytes": 64,
  "sequence": 2,
  "ttl": 63
}
{
  "source": "172.31.255.0",
  "time": "1001000000",
  "sent": 2,
  "received": 2,
  "minTime": "25210000",
  "avgTime": "27510000",
  "maxTime": "29810000",
  "stdDev": "2300000"
}
```
```
$ grpcurl -H 'username: arista'  -H 'password: arista' -d '{"destination": "172.31.255.0", "max_ttl": 50, "do_not_resolve":true}' -import-path ${GOPATH}/src -proto github.com/openconfig/gnoi/system/system.proto -plaintext 10.73.1.118:6030 gnoi.system.System/Traceroute
{
  "destinationName": "172.31.255.0",
  "destinationAddress": "172.31.255.0",
  "hops": 50,
  "packetSize": 60
}
{
  "hop": 1,
  "address": "10.90.90.1",
  "rtt": "16589000"
}
{
  "hop": 1,
  "address": "10.90.90.1",
  "rtt": "17886000"
}
{
  "hop": 1,
  "address": "10.90.90.1",
  "rtt": "23219000"
}
{
  "hop": 2,
  "address": "172.31.255.0",
  "rtt": "46537000"
}
{
  "hop": 2,
  "address": "172.31.255.0",
  "rtt": "47873000"
}
{
  "hop": 2,
  "address": "172.31.255.0",
  "rtt": "55376000"
}
```
