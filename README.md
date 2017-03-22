# Shadowsocks-libev
Operating system:	Centos 7 x86_64 minimal  
# 系统升级

```
yum install deltarpm epel-release
yum update
yum install git net-tools iptables-services policycoreutils gettext gcc autoconf libtool automake make asciidoc xmlto udns-devel libev-devel pcre-devel
```

# Installation of Libsodium

```
export LIBSODIUM_VER=1.0.12
wget https://download.libsodium.org/libsodium/releases/libsodium-$LIBSODIUM_VER.tar.gz
tar xvf libsodium-$LIBSODIUM_VER.tar.gz
pushd libsodium-$LIBSODIUM_VER
./configure --prefix=/usr && make
make install
popd
ldconfig
```
# Installation of MbedTLS

```
export MBEDTLS_VER=2.4.2
wget https://tls.mbed.org/download/mbedtls-$MBEDTLS_VER-gpl.tgz
tar xvf mbedtls-$MBEDTLS_VER-gpl.tgz
pushd mbedtls-$MBEDTLS_VER
make SHARED=1 CFLAGS=-fPIC
make DESTDIR=/usr install
popd
ldconfig
```
# Installation of Shadowsocks-libev
```
git clone https://github.com/shadowsocks/shadowsocks-libev.git
cd shadowsocks-libev
git submodule update --init --recursive
./autogen.sh && ./configure --disable-documentation && make
make install
popd
ldconfig
```

## Run with log
<pre>
nohup ss-server -p 443 -k password -m rc4-md5 -A -a shadowsocks -v >>/tmp/ss-443.log 2>&1 &
</pre>

## Server-multi-port

server-multi-port.json
<pre>
{
	"port_password": {
		"8387": "foobar",
		"8388": "barfoo"
	},
	"method": "aes-128-cfb",
	"timeout": 600
}
</pre>
### Run server-multi-port
<pre>
nohup ss-manager --manager-address /var/run/shadowsocks-manager.sock -A -c /server-multi-port.json &
</pre>
# Iptables
## 安装iptables services
<pre>
yum install net-tools iptables-services policycoreutils
</pre>
## 清除 iptables 规则
<pre>
iptables -t filter -F
iptables -t filter -X
iptables -t filter -Z
iptables -t mangle -F
iptables -t mangle -X
iptables -t mangle -Z
iptables -t nat -F
iptables -t nat -X
iptables -t nat -Z
iptables -t raw -F
iptables -t raw -X
iptables -t raw -Z
</pre>
## 禁止BT
<pre>
iptables -A FORWARD -m string --string "GET /scrape?info_hash=" --algo bm --to 65535 -j DROP
iptables -A FORWARD -m string --string "GET /announce.php?info_hash=" --algo bm --to 65535 -j DROP
iptables -A FORWARD -m string --string "GET /scrape.php?info_hash=" --algo bm --to 65535 -j DROP
iptables -A FORWARD -m string --string "GET /scrape.php?passkey=" --algo bm --to 65535 -j DROP
iptables -A FORWARD -m string --hex-string "|13426974546f7272656e742070726f746f636f6c|" --algo bm --to 65535 -j DROP
</pre>
## Iptables rules for SHADOWSOCKS
<pre>
iptables -N SHADOWSOCKS
iptables -t filter -I SHADOWSOCKS -p tcp --syn -m connlimit --connlimit-above 40 -j REJECT --reject-with tcp-reset
iptables -t filter -A SHADOWSOCKS -d 127.0.0.0/8 -j REJECT
iptables -t filter -A SHADOWSOCKS -d 0.0.0.0/8 -j REJECT
iptables -t filter -A SHADOWSOCKS -d 10.0.0.0/8 -j REJECT
iptables -t filter -A SHADOWSOCKS -d 169.254.0.0/16 -j REJECT
iptables -t filter -A SHADOWSOCKS -d 172.16.0.0/12 -j REJECT
iptables -t filter -A SHADOWSOCKS -d 192.168.0.0/16 -j REJECT
iptables -t filter -A SHADOWSOCKS -d 224.0.0.0/4 -j REJECT
iptables -t filter -A SHADOWSOCKS -d 240.0.0.0/4 -j REJECT
iptables -t filter -A SHADOWSOCKS -p udp --dport 53 -j ACCEPT
iptables -t filter -A SHADOWSOCKS -p tcp --dport 53 -j ACCEPT
iptables -t filter -A SHADOWSOCKS -p tcp --dport 80 -j ACCEPT
iptables -t filter -A SHADOWSOCKS -p tcp --dport 443 -j ACCEPT
iptables -t filter -A SHADOWSOCKS -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -t filter -A SHADOWSOCKS -p tcp -j REJECT --reject-with tcp-reset
iptables -t filter -A SHADOWSOCKS -p udp -j REJECT
iptables -A OUTPUT -j SHADOWSOCKS
</pre>
### Run as user
```
useradd -s /usr/sbin/nologin -r -m -d /shadowsocks shadowsocks
```
```
iptables -N SHADOWSOCKS
iptables -t filter -m owner --uid-owner shadowsocks -I SHADOWSOCKS -p tcp --syn -m connlimit --connlimit-above 32 -j REJECT --reject-with tcp-reset
iptables -t filter -m owner --uid-owner shadowsocks -A SHADOWSOCKS -d 127.0.0.0/8 -j REJECT
iptables -t filter -m owner --uid-owner shadowsocks -A SHADOWSOCKS -d 0.0.0.0/8 -j REJECT
iptables -t filter -m owner --uid-owner shadowsocks -A SHADOWSOCKS -d 10.0.0.0/8 -j REJECT
iptables -t filter -m owner --uid-owner shadowsocks -A SHADOWSOCKS -d 169.254.0.0/16 -j REJECT
iptables -t filter -m owner --uid-owner shadowsocks -A SHADOWSOCKS -d 172.16.0.0/12 -j REJECT
iptables -t filter -m owner --uid-owner shadowsocks -A SHADOWSOCKS -d 192.168.0.0/16 -j REJECT
iptables -t filter -m owner --uid-owner shadowsocks -A SHADOWSOCKS -d 224.0.0.0/4 -j REJECT
iptables -t filter -m owner --uid-owner shadowsocks -A SHADOWSOCKS -d 240.0.0.0/4 -j REJECT
iptables -t filter -m owner --uid-owner shadowsocks -A SHADOWSOCKS -p udp --dport 53 -j ACCEPT
iptables -t filter -m owner --uid-owner shadowsocks -A SHADOWSOCKS -p tcp --dport 53 -j ACCEPT
iptables -t filter -m owner --uid-owner shadowsocks -A SHADOWSOCKS -p tcp --dport 80 -j ACCEPT
iptables -t filter -m owner --uid-owner shadowsocks -A SHADOWSOCKS -p tcp --dport 443 -j ACCEPT
iptables -t filter -m owner --uid-owner shadowsocks -A SHADOWSOCKS -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -t filter -m owner --uid-owner shadowsocks -A SHADOWSOCKS -p tcp -j REJECT --reject-with tcp-reset
iptables -t filter -m owner --uid-owner shadowsocks -A SHADOWSOCKS -p udp -j REJECT
iptables -A OUTPUT -j SHADOWSOCKS
```
### kcptun
```
nohup kcptun -t 127.0.0.1:443 -l :1234 -mode normal -crypt none -nocomp -dscp 46 -parityshard 0 -sndwnd 300 -rcvwnd 300 &
nohup ss-server -s 127.0.0.1 -p 443 -k password -m rc4-md5 -A -v >>/root/ss-443.log 2>&1 &
```
#### Iptables rules for kcptun
```
iptables -t filter -m owner --uid-owner shadowsocks -I SHADOWSOCKS -p tcp -s 127.0.0.1 -d 127.0.0.1 --sport 443 -j ACCEPT
```
## iptables save
<pre>
service iptables save
service iptables start
service iptables stop
service iptables restart
systemctl enable iptables.service
</pre>

