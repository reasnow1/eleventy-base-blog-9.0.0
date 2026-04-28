---
title: Alpine linux安装rtorrent Alpine linux install rtorren
description: This is a post on My Blog about leveraging agile frameworks.
date: 2026-04-27
tags: Alpine linux
---
Alpine linux安装rtorrent

确保镜像源已经弄好

使用root

```bash
apk update
apk add rtorrent
apk add screen
apk add mediainfo
```

使用普通用户

```bash
vi .rtorrent.rc
```

```config
#############################################################################
# A minimal rTorrent configuration that provides the basic features
# you want to have in addition to the built-in defaults.
#
# See https://github.com/rakshasa/rtorrent/wiki/CONFIG-Template
# for an up-to-date version.
#############################################################################


## Instance layout (base paths)
method.insert = cfg.basedir,  private|const|string, (cat,"/home/xinya/rtorrent/")
method.insert = cfg.download, private|const|string, (cat,(cfg.basedir),"download/")
method.insert = cfg.logs,     private|const|string, (cat,(cfg.basedir),"log/")
method.insert = cfg.logfile,  private|const|string, (cat,(cfg.logs),"rtorrent-",(system.time),".log")
method.insert = cfg.session,  private|const|string, (cat,(cfg.basedir),".session/")
method.insert = cfg.watch,    private|const|string, (cat,(cfg.basedir),"watch/")


## Create instance directories
execute.throw = sh, -c, (cat,\
    "mkdir -p \"",(cfg.download),"\" ",\
    "\"",(cfg.logs),"\" ",\
    "\"",(cfg.session),"\" ",\
    "\"",(cfg.watch),"/load\" ",\
    "\"",(cfg.watch),"/start\" ")


## Listening port for incoming peer traffic (fixed; you can also randomize it)
network.port_range.set = 50000-50000
network.port_random.set = no


## Tracker-less torrent and UDP tracker support
## (conservative settings for 'private' trackers, change for 'public')
dht.mode.set = auto
protocol.pex.set = yes
dht.port.set = 6881
trackers.use_udp.set = yes


## Peer settings
throttle.max_uploads.set = 100
throttle.max_uploads.global.set = 250

throttle.min_peers.normal.set = 20
throttle.max_peers.normal.set = 60
throttle.min_peers.seed.set = 30
throttle.max_peers.seed.set = 80
trackers.numwant.set = 80

protocol.encryption.set = allow_incoming,try_outgoing,enable_retry


## Limits for file handle resources, this is optimized for
## an `ulimit` of 1024 (a common default). You MUST leave
## a ceiling of handles reserved for rTorrent's internal needs!
network.http.max_open.set = 50
network.max_open_files.set = 600
network.max_open_sockets.set = 300


## Memory resource usage (increase if you have a large number of items loaded,
## and/or the available resources to spend)
pieces.memory.max.set = 1800M
network.xmlrpc.size_limit.set = 4M


## Basic operational settings (no need to change these)
session.path.set = (cat, (cfg.session))
directory.default.set = (cat, (cfg.download))
log.execute = (cat, (cfg.logs), "execute.log")
#log.xmlrpc = (cat, (cfg.logs), "xmlrpc.log")
execute.nothrow = sh, -c, (cat, "echo >",\
    (session.path), "rtorrent.pid", " ",(system.pid))


## Other operational settings (check & adapt)
encoding.add = UTF-8
system.umask.set = 0027
system.cwd.set = (directory.default)
network.http.dns_cache_timeout.set = 25
schedule2 = monitor_diskspace, 15, 60, ((close_low_diskspace, 1000M))
#pieces.hash.on_completion.set = no
#view.sort_current = seeding, greater=d.ratio=
#keys.layout.set = qwerty
#network.http.capath.set = "/etc/ssl/certs"
#network.http.ssl_verify_peer.set = 0
#network.http.ssl_verify_host.set = 0
#network.rpc.use_xmlrpc.set = true
#network.rpc.use_jsonrpc.set = true

## Some additional values and commands
method.insert = system.startup_time, value|const, (system.time)
method.insert = d.data_path, simple,\
    "if=(d.is_multi_file),\
        (cat, (d.directory), /),\
        (cat, (d.directory), /, (d.name))"
method.insert = d.session_file, simple, "cat=(session.path), (d.hash), .torrent"


## Watch directories (add more as you like, but use unique schedule names)
## Add torrent
schedule2 = watch_load, 11, 10, ((load.verbose, (cat, (cfg.watch), "load/*.torrent")))
## Add & download straight away
schedule2 = watch_start, 10, 10, ((load.start_verbose, (cat, (cfg.watch), "start/*.torrent")))


## Run the rTorrent process as a daemon in the background
## (and control via XMLRPC sockets)
#system.daemon.set = true
#network.scgi.open_local = /home/xinya/rtorrent/rpc.socket 
#execute.nothrow = chmod,770,(cat,(session.path),rpc.socket)
network.scgi.open_port = 192.168.18.210:5000


## Logging:
##   Levels = critical error warn notice info debug
##   Groups = connection_* dht_* peer_* rpc_* storage_* thread_* tracker_* torrent_*
print = (cat, "Logging to ", (cfg.logfile))
log.open_file = "log", (cfg.logfile)
log.add_output = "info", "log"
#log.add_output = "tracker_debug", "log"

network.bind_address.ipv6.set = 输入公网ipv6地址
#network.prefer.ipv6.set = true
network.bind_address.ipv4.set = 0.0.0.0
### END of rtorrent.rc ###
```

修改部分：

- 开头修改路径
- 打开DHT，UDP
- 增加SCGI端口，IP地址使用远控电脑的
- 注：作者要求使用rtorrent.sock更安全，不过，我懒得理了，直接http。
- 必须加后面这里才能同时ipv4和6。

现在打开rtorrent:

```bash
screen
rtorrent
```

按Ctrl+a，然后D脱离。

回去方法：

```bash
screen -ls
There is a screen on:
	3065.pts-0.localhost	(Detached)
1 Socket in /home/用户名/.screen.
screen -r 3065
```

# 安装Flood

Debian上操作：

```bash
export http_proxy=http://127.0.0.1:7897
export https_proxy=http://127.0.0.1:7897
# Download and install nvm:
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.4/install.sh | bash

# in lieu of restarting the shell
\. "$HOME/.nvm/nvm.sh"

# Download and install Node.js:
nvm install 24

# Verify the Node.js version:
node -v # Should print "v24.15.0".

# Verify npm version:
npm -v # Should print "11.12.1".
```

其实就是安装Node，也可以看
https://nodejs.org/en/download

```bash
@debian:~/alpine$ npm install --global flood

added 1 package in 4s
npm notice
npm notice New minor version of npm available! 11.12.1 -> 11.13.0
npm notice Changelog: https://github.com/npm/cli/releases/tag/v11.13.0
npm notice To update run: npm install -g npm@11.13.0
npm notice
@debian:~/alpine$ flood
{"level":30,"time":"2026-04-25T06:59:15.075Z","pid":30688,"hostname":"debian","name":"flood:web-server","msg":"Server listening at http://127.0.0.1:3000"}
{"level":30,"time":"2026-04-25T06:59:15.076Z","pid":30688,"hostname":"debian","name":"flood:web-server","url":"http://127.0.0.1:3000","version":"4.13.9","msg":"Flood server listening"}
```

现在可以浏览器打开http://127.0.0.1:3000了。

输入服务器IP，端口5000即可连接。

防火墙：

```config
config rule
	option src 'wan'
	option dest 'lan'
	option name 'allow-rtorrent'
	option dest_port '50000'
	option target 'ACCEPT'
	list dest_ip '192.168.18.210'
	list dest_ip '公网IPV6'

config rule
	option src 'wan'
	option dest 'lan'
	option name 'allow-rtorrent-dht'
	option dest_port '6881'
	option target 'ACCEPT'
	list dest_ip '192.168.18.210'
	list dest_ip '公网IPV6'
```