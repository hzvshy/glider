# glider
glider is a forward proxy with multiple protocols support.

we can set up local listeners as proxy, and forward requests to internet via forwarders.
```
                |Forwarder ----------------->|         
   Listener --> |                            | Internet
                |Forwarder --> Forwarder->...| 
```

## Features
Listen(local proxy):
- Socks5 proxy
- Http proxy
- SS proxy
- Linux transparent proxy(iptables redirect)
- TCP tunnel
- DNS tunnel(udp2tcp)

Forward(upstream proxy):
- Socks5 proxy
- Http proxy
- SS proxy

General:
- Forward chain
- HA or RR strategy for multiple forwarders
- Http and socks5 on the same port
- Periodical proxy checking

## Install
Binary: 
- [https://github.com/nadoo/glider/releases](https://github.com/nadoo/glider/releases)

Go Get :
```bash
go get -u github.com/nadoo/glider
```

ArchLinux: 
```bash
sudo pacman -S glider
```

## Run
command line:
```bash
glider -listen :8443 -verbose
```

config file:
```bash
glider -config CONFIGPATH
```

command line with config file:
```bash
glider -config CONFIGPATH -listen :8080 -verbose
```

## Usage
```bash
glider v0.2 usage:
  -checkduration int
        proxy check duration(seconds) (default 30)
  -checkhost string
        proxy check address (default "www.apple.com:443")
  -config string
        config file path
  -forward value
        forward url, format: SCHEMA://[USER|METHOD:PASSWORD@][HOST]:PORT[,SCHEMA://[USER|METHOD:PASSWORD@][HOST]:PORT]
  -listen value
        listen url, format: SCHEMA://[USER|METHOD:PASSWORD@][HOST]:PORT
  -strategy string
        forward strategy, default: rr (default "rr")
  -verbose
        verbose mode

Available Schemas:
  mixed: serve as a http/socks5 proxy on the same port. (default)
  ss: ss proxy
  socks5: socks5 proxy
  http: http proxy
  redir: redirect proxy. (used on linux as a transparent proxy with iptables redirect rules)
  tcptun: a simple tcp tunnel
  dnstun: listen on udp port and forward all dns requests to remote dns server via forwarders(tcp)

Available schemas for different modes:
  listen: mixed ss socks5 http redir tcptun dnstun
  forward: ss socks5 http

Available methods for ss:
  AEAD_AES_128_GCM AEAD_AES_192_GCM AEAD_AES_256_GCM AEAD_CHACHA20_POLY1305 AES-128-CFB AES-128-CTR AES-192-CFB AES-192-CTR AES-256-CFB AES-256-CTR CHACHA20-IETF XCHACHA20
  NOTE: chacha20-ietf-poly1305 = AEAD_CHACHA20_POLY1305

Available forward strategies:
  rr: Round Robin mode
  ha: High Availability mode

Config file format(see `glider.conf.example` as an example):
  # COMMENT LINE
  KEY=VALUE
  KEY=VALUE
  # KEY equals to command line flag name: listen forward strategy...

Examples:
  glider -config glider.conf
    -run glider with specified config file.

  glider -listen :8443
    -listen on :8443, serve as http/socks5 proxy on the same port.

  glider -listen ss://AEAD_CHACHA20_POLY1305:pass@:8443
    -listen on 0.0.0.0:8443 as a ss server.

  glider -listen socks5://:1080 -verbose
    -listen on :1080 as a socks5 proxy server, in verbose mode.

  glider -listen http://:8080 -forward socks5://127.0.0.1:1080
    -listen on :8080 as a http proxy server, forward all requests via socks5 server.

  glider -listen redir://:1081 -forward ss://method:pass@1.1.1.1:443
    -listen on :1081 as a transparent redirect server, forward all requests via remote ss server.

  glider -listen tcptun://:80=2.2.2.2:80 -forward ss://method:pass@1.1.1.1:443
    -listen on :80 and forward all requests to 2.2.2.2:80 via remote ss server.

  glider -listen socks5://:1080 -listen http://:8080 -forward ss://method:pass@1.1.1.1:443
    -listen on :1080 as socks5 server, :8080 as http proxy server, forward all requests via remote ss server.

  glider -listen redir://:1081 -listen dnstun://:53=8.8.8.8:53 -forward ss://method:pass@server1:port1,ss://method:pass@server2:port2
    -listen on :1081 as transparent redirect server, :53 as dns server, use forward chain: server1 -> server2.

  glider -listen socks5://:1080 -forward ss://method:pass@server1:port1 -forward ss://method:pass@server2:port2 -strategy rr
    -listen on :1080 as socks5 server, forward requests via server1 and server2 in roundrbin mode.
```

## Config File 
Command:
```bash
glider -config glider.conf
```
Config file, **just use the command line flag name as the key name**:
```bash
### glider config file

# verbose mode, print logs
verbose

# listen on 8443, serve as http/socks5 proxy on the same port.
listen=:8443

# listen on udp port 53, forward dns requests via tcp protocol
listen=dnstun://:53=8.8.8.8:53

# upstream forward proxy
forward=socks5://192.168.1.10:1080

# upstream forward proxy
forward=ss://method:pass@1.1.1.1:443

# upstream forward proxy (forward chain)
forward=http://1.1.1.1:8080,socks5://2.2.2.2:1080

# multiple upstream proxies forwad strategy
strategy=rr

# check address (to check whether a host is reachable via forward proxy)
checkhost=www.apple.com:443

# check duration
checkduration=30
```
See [glider.conf.example](glider.conf.example)

## Service
- systemd: [https://github.com/nadoo/glider/blob/master/systemd/](https://github.com/nadoo/glider/blob/master/systemd/)

## Links
- [go-ss2](https://github.com/shadowsocks/go-shadowsocks2): the core ss protocol support
- [gost](https://github.com/ginuerzh/gost): ideas and inspirations
- [conflag](https://github.com/nadoo/conflag): command line and config file parse support
- [ArchLinux](https://www.archlinux.org/packages/community/x86_64/glider): a great linux distribution with glider pre-built package
