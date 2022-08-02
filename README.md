# Cloud Interconnection Benchmarking Setup

#### 1.	Follow the [setup document](https://github.com/equinix/cloud-interconnection-benchmarking-backup/blob/master/docs/Cloud%20Interconnection%20Benchmarking%20Backup%20-%20Infrastructure%20Configuration%20v1.0.pdf) to create the environment.


#### 2.	Commands executed before each network path viz. Internet, FastConnect private peering, FastConnect private peering with 1500 MTU and FastConnect public peering
$# shows the MTU configured on the Oracle DB box
```jsx
ifconfig| grep mtu
```
$# shows the effective MTU to OCI object storage in San Jose region
```jsx
ping objectstorage.us-sanjose-1.oraclecloud.com -c 2 -M do -s 8900
```
$# Shows the traceroute. Over private peering this would be lesser than # that for internet
```jsx
traceroute objectstorage.us-sanjose-1.oraclecloud.com
```
$# Shows the RTT using ICMP ping
```jsx
ping objectstorage.us-sanjose-1.oraclecloud.com -c 10
```

Commands for setting the MTU to 1500 for the private peering
```jsx
$# setting MTU as 1500
ifconfig bond0 mtu 1500 up
ifconfig bond0.1060 mtu 1500 up
ifconfig bond0.1003 mtu 1500 up
ifconfig| grep mtu
```

#### 3.	Now for each network path, Navigate to the Backup script (scripts/database/) location

For the first execution use the following command to ensure that the backups are going through and getting deleted at the end of the script. This can be observed by browsing the bucket using the OCI console.
```jsx
nohup ./backupndelete.sh 0 0 &
```
Populate the input.txt with the list of latency and loss combinations. Each combination need to be on a new line. To help with this a sample input.txt can be found at “input.txt.full” file.

Once successful, Kick off the loop script with the command 
```jsx
nohup ./ loop.backupndelete.sh &
```

Following log files are outputted
*	benchmark.log has the primary stdout
*	The output from each combination can be found under logs/Test_<latency>_<loss>.txt file
*	The rman logs for backup can be found at /home/oracle/script_log/fullbkp.log
*	The rman logs for deleting the backup can be found at /home/oracle/script_log/backup_del.log

#### 4.	After the script has executed, the backup times can be captured by going into the scripts/database/stored-procedures/ directory and entering the sqlplus interface and running the “@chkbkp1” command

```jsx
SQL> @chkbkp1
4 DB FULL       COMPLETED 05/31/22 11:08 05/31/22 11:24       16.4
10 DB FULL       COMPLETED 05/31/22 13:55 05/31/22 14:10       15.6
18 DB FULL       COMPLETED 05/31/22 14:13 05/31/22 14:28       15.5
26 DB FULL       COMPLETED 05/31/22 14:31 05/31/22 14:47       15.7
```
This will output all the backup times of the previous and current backups. At the end of each line, it outputs the number of minutes each backup took.

#### 5.	Sometimes it may be useful to reset TC’s loss / latency setting. Following command was used to reset the TC’s setting on the TC box:
```jsx
"tc qdisc show | tail -2" 
"tc qdisc del dev bond1.1060 root"
"tc qdisc del dev bond1.1003 root"
"tc qdisc show | tail -2"
```

