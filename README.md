## About redis2bsm
**OS**: CentOS 6.6 +

**Usecase**: 
redis2bsm was developed for the usecase to catch specific syslog events sent to a ELK installation. The only way to get notified by a specific error from some crappy network equipment was with syslog messages. To automagicly get notified about these events a logstash instance identifies and forward these to a redis queue. Then this redis2bsm daemon pops that redis queue and forwards events to a soap api on a HP BSM Connector instance. 

### INSTALL
#### prerequisite 
**- python dependencies**:
```
yum install python-simplejson python-redis
```
**- redis**
```
yum install redis
```
*example redis conf*:
```
cat <<EOF | tee /etc/redis.conf
daemonize yes
pidfile /var/run/redis/redis.pid
port 6379
bind 127.0.0.1
timeout 0
tcp-keepalive 60
loglevel notice
syslog-enabled yes
syslog-ident redis
syslog-facility local0
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename redis_db_dump.rdb
dir /var/lib/redis
slave-serve-stale-data yes
slave-read-only yes
repl-disable-tcp-nodelay no
slave-priority 100
maxmemory 512M
maxmemory-policy volatile-lru
appendonly no
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
lua-time-limit 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-entries 512
list-max-ziplist-value 64
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
aof-rewrite-incremental-fsync yes
EOF
```

#### redis2bsm

```
mkdir -p /opt/redis2bsm && cd /opt/redis2bsm
git clone https://github.com/j0nix/redis2bsm.git
cp redis2bsm.conf.example /etc/redis2bsm.conf
cp redis2bsm.init /etc/init.d/redis2bsm && chkconfig --add redis2bsm
cp redis2bsm.logrotate /etc/logrotate.d/redis2bsm
```
**Before proceed, Make changes in /etc/redis2bsm.conf according to your enviroment**


### Example of Logstash conf pushing events to redis 
```
input {
        udp {
            port => 10514
        }
}

filter {
        json {
          source => "message"
        }
        # Verify that we have a ALARM field
        if [ALARM] {
                mutate {
                    #Fields for notifying
                    add_field => {
                        "[@alarm][title]" => "%{ALARM} - System process crashed"
                        "[@alarm][description]" => "%{message}"
                        "[@alarm][severity]" => "CRITICAL"
                        "[@alarm][subcategory]" => "%{facility}"
                        "[@alarm][node]" => "%{host}"
                        "[@alarm][ci_hint]" => "%{location}"
                        "[@alarm][application]" => "%{ALARM}"
                        "[@alarm][object]" => "syslog"
                    }
                    add_tag => [ "forwardToBSM" ]
                }
        }
        else {
                drop { }
        }
}
output {
        redis {
            host => ['127.0.0.1']
            data_type => "list"
            key => "ALARM"
        }
}
```
