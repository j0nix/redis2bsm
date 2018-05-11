## About redis2bsm
Fetches data from a redis queue, preferably with data pushed from logstash, and forwards to HP BSM connector 

## INSTALL
python-simplejson
python-redis

# Create dir	
mkdir /opt/redis2bsm
mv daemon /opt/redis2bsm

### Conf
/etc/redis2bsm.conf
	
### homebrewed init script
/etc/init.d/redis2bsm
chkconfig --add redis2bsm

### logrotate
/etc/logrotate.d/redis2bsm":
