# Mac清除DNS缓存

Mac OS X下清除DNS缓存的命令是：
yosemite：
    

```shell
sudo discoveryutil mdnsflushcache
```

Lion、Mountain Lion和Mavericks：
    

```shell
sudo killall -HUP mDNSResponder
```

Leopard和Snow Leopard：
    

```shell
sudo dscacheutil -flushcache
```

Tiger或更低版本 Mac OS：
    

```shell
sudo lookupd -flushcache
```

