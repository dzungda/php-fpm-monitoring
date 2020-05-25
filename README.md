# php-fpm-monitoring

![](php-fpm.png)

## php-fpm installation


## influxdb installation 
(https://docs.influxdata.com/influxdb/v1.5/introduction/installation/)

Debian 9
```hcl
curl -sL https://repos.influxdata.com/influxdb.key | sudo apt-key add -
echo "deb https://repos.influxdata.com/debian stretch stable" > /etc/apt/sources.list.d/influxdata.list
apt-get update
apt-get install influxdb
systemctl start influxdb
```
###Configure InfluxDB
```hcl
root@server ~# influx
Connected to http://localhost:8086 version 1.5.1
InfluxDB shell version: 1.5.1
```
Create the database
```hcl
> CREATE DATABASE telegraf
> SHOW DATABASES
name: databases
name
----
_internal
telegraf
```
Create a user. Choose a good password as InfluxDB will be exposed to the internet
```hcl
> CREATE USER telegraf WITH PASSWORD 'password'
> GRANT ALL ON telegraf TO telegraf
> SHOW USERS;
user     admin
----     -----
telegraf false
```
You can setup a retention policy if you wish

```hcl
> CREATE RETENTION POLICY thirty_days ON telegraf DURATION 30d REPLICATION 1 DEFAULT
> SHOW RETENTION POLICIES ON telegraf
name		duration	replicaN	DEFAULT
DEFAULT		0		1		FALSE
thirty_days	720h0m0s	1		TRUE
```
## telegraf installation 
(https://docs.influxdata.com/telegraf/v1.14/introduction/installation/)

### Configure Telegraf

I suggest you to read it, but here's a quick start on what you can add in /etc/telegraf/telegraf.conf.
Agent configuration
```hcl
[agent]
  hostname = "myserver"
  flush_interval = "15s"
  interval = "15s"
```
By default, the hostname will be the server hostname (makes sense), and the metrics will be collected every 10 seconds.

Inputs configuration

```hcl
[agent]
  hostname = "myserver"
  flush_interval = "15s"
  interval = "15s"

[[inputs.cpu]]

[[inputs.mem]]

[[inputs.system]]

[[inputs.disk]]
  mount_points = ["/"]

[[inputs.processes]]

[[inputs.net]]
  fieldpass = [ "bytes_*" ]
  
[[inputs.phpfpm]]
  urls = ["fcgi://127.0.0.1:9000/status"]
  
[[outputs.influxdb]]
  database = "telegraf"
  urls = [ "http://127.0.0.1:8086" ]
  username = "telegraf"
  password = "scloud!"

```
