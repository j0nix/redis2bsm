## About redis2bsm
Fetches data from a redis queue, preferably with data pushed from logstash, and forwards to HP BSM connector 

## INSTALL
yum install python-simplejson
yum install python-redis
mkdir /opt/redis2bsm
mv redis2bsm classes /opt/redis2bsm

### CONF
/etc/redis2bsm.conf
	
### Homebrewed init script
/etc/init.d/redis2bsm
chkconfig --add redis2bsm

### logrotate
/etc/logrotate.d/redis2bsm
