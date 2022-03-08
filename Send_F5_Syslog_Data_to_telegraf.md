# Send F5 Syslog Data to Telegraf

This article shows how to collect syslog data into InfluxDB using telegraf and build a Grafana dashboard

![flow_telegraf](assets/flow_telegraf.png)

## Preface

### Installed versions
* Debian 11 bullseye
* rsyslog 8.2102.0-2
* Grafana 8.4.3
* InfluxDB 1.6.7~rc0-1+b5
* telegraf 1.21.4-1

### Diagram


## Install prerequisites
```shell
root@rsyslog:~# apt install curl apt-transport-https software-properties-common wget gnupg
```

## Step 1: Install Grafana OSS release
Official Grafana Guide: [Install Grafana on Debian or Ubuntu](https://grafana.com/docs/grafana/labigip/installation/debian/)
```shell
root@rsyslog:~# echo "deb https://packages.grafana.com/oss/deb stable main" | tee -a /etc/apt/sources.list.d/grafana.list
root@rsyslog:~# wget -q -O - https://packages.grafana.com/gpg.key | apt-key add -
root@rsyslog:~# apt update
root@rsyslog:~# apt install grafana
```

## Step 2: Install InfluxDB 1.x Open Source

### Install InfluxDB 1.x Open Source
```shell
root@rsyslog:~# wget https://dl.influxdata.com/influxdb/releases/influxdb_1.8.10_amd64.deb
root@rsyslog:~# dpkg -i influxdb_1.8.10_amd64.deb
```
### Configure InfluxDB
Enable the HTTP endpoint by uncommenting the following four lines in __/etc/influxdb/influxdb.conf__.

```text
[http]
  # Determines whether HTTP endpoint is enabled.
    enabled = true

  # The bind address used by the HTTP service.
    bind-address = ":8086"

  # Determines whether user authentication is enabled over HTTP/HTTPS.
    auth-enabled = false

  # The default realm sent back when issuing a basic auth challenge.
  # realm = "InfluxDB"

  # Determines whether HTTP request logging is enabled.
    log-enabled = true
```

### Add InfluxDB user with administrative permissions and create a database
influx
CREATE USER <<yourusername>> WITH PASSWORD <<yourpassword>> WITH ALL PRIVILEGES
CREATE DATABASE <<yourdatabasename>>
exit

## Step 3: Install Telegraf

### Install Telegraf
```shell
root@rsyslog:~# wget https://dl.influxdata.com/telegraf/releases/telegraf_1.21.4-1_amd64.deb
root@rsyslog:~# sudo dpkg -i telegraf_1.21.4-1_amd64.deb
```

### Configure Telegraf /etc/telegraf/telegraf.conf
Output to InfluxDB
```text
[[outputs.influxdb]]
  urls = ["http://127.0.0.1:8086"]
  database = <<yourdatabasename>>
  username = <<yourusername>>
  password = <<yourpassword>>
```

Input from Syslog
```text
[[inputs.syslog]]
  server = "udp://:6514"
```

## Step 4: Install Rsyslog

### Configure /etc/rsyslog.conf
Uncomment
```text
# provides UDP syslog reception
module(load="imudp")
input(type="imudp" port="514")
```

Add
```text
$template remote-incoming-logs,"/var/log/%HOSTNAME%/%PROGRAMNAME%.log"
*.* ?remote-incoming-logs
```

### /etc/rsyslog.d/50-telegraf.conf
```text
$ActionQueueType LinkedList # use asynchronous processing
$ActionQueueFileName srvrfwd # set file name, also enables disk mode
$ActionResumeRetryCount -1 # infinite retries on insert failure
$ActionQueueSaveOnShutdown on # save in-memory data if rsyslog shuts down

:hostname, contains, "afm"
*.* @127.0.0.1:6514;RSYSLOG_SyslogProtocol23Format
```

## Step 5: Configure remote logging on F5

## Step 6: Confirm data is flowing in

## Step 7: Create Grafana dashboards