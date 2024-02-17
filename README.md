# telegraf-drbd_collector
Basic script for use with the telegraf exec input plugin to collect DRBD status data into influxdb for display in a grafana dashboard.

### Configuration
#### v1
```
[[inputs.exec]]
  commands = [
    "/usr/local/bin/drbd_collector"
  ]
  timeout = "5s"
  name_suffix = ""
  data_format = "influx"
```
#### v2
```
[[inputs.exec]]
  commands = [
    "/usr/local/bin/drbd_collector-v2 --statistics"
  ]
  timeout = "5s"
  name_suffix = ""
  data_format = "influx"
```

### Sudoers
By default the telegraf service runs as the telegraf user which will prevent the drbd_collector script from running the drdbadm command.
Add the following to the suders.conf file to permit sudo access:

```telegraf ALL=(ALL) NOPASSWD: /usr/sbin/drbdsetup```

To prevent log spam from sudo access also add the following:

```Defaults:telegraf !syslog```

### Metrics
- state
- role
- disk
- resync

#### Optional Metrics
When using the v2 script and the ```--statistics``` parameter these additional metrics will be provided per volume
- read
- written
- sent
- received

#### Tags
- resource
- volume

#### Example Output
```
drbd,host=node04,resource=nfs,volume=0 disk=0i,role=2i,state=8i 1708126124000000000
drbd,host=node04,resource=nfs,volume=1 disk=0i,role=2i,state=8i 1708126124000000000
drbd,host=node04,resource=nfs,volume=2 disk=0i,role=2i,state=8i 1708126124000000000
```

#### Notes
The state, role, and disk metrics are sent to influx as integer values rather than strings. This permits Grafana dashboard panels
to make use of Thresholds for cell coloring along with use in alerting. You can use Value Mappings within the Grafana panel to
convert back to text values for display purposes.

The resync metric is only provided in the output when it is present in the drbdadm status.

##### ToDO
Potentially rewrite in go for inclusion as telegraf plugin.
