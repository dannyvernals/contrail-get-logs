# Contrail Get Logs
Quick and fairly simple script to grab log files from Contrail Components.  
There are more advanced tools around, such as:  
https://github.com/Juniper/contrail-controller/blob/master/src/config/utils/vrouter_agent_debug_tool.py

However they are not always suitable.  For example, the above only gets logs off vrouters and Contrail Controllers not components such as Analytics etc.  It also requires HTTP connectivity to Introspect endpoints which is not always available.

This script attempts to fill that gap.  It grabs all the logs from the specified directories / files listed in the YAML config file.  An example config file is included in the repo.
If indicated in the config, it will also grab the logs from all running containers i.e. the JSON version of 'docker logs <container-name>'

If requested it can also remove confidential data from the log files before sharing with a: vendor, supplier or team.

Only python3 is supported.

## Example usage

### 1) Basic operation, supply remote device IP as an argument
usage: ./contrail_get_logs.py config-file component-name -d IP/FQDN

```
python3 ./contrail_get_logs.py get_log_config.yaml vrouter -d 172.16.0.134
zipping '/var/log/contrail/' up to /var/tmp/_var_log_contrail_.tgz on 172.16.0.134
grabbing /var/tmp/_var_log_contrail_.tgz
_var_log_contrail_.tgz                                                                                              100% 1651KB  49.0MB/s   00:00    
connecting to '172.16.0.134' to get container logs
getting logs for container 'vrouter_nodemgr_1'
getting logs for container 'vrouter_vrouter-agent_1'
cleaning up temporary files and zipping up to 'contrail-agent-1b2fec5c-d014-11ea-8ad0-1ede62014782-logs.tgz'
```

### 2) Basic operation, use an IPs config file
Instead of specifying IP / hostname as an argument you can use a pre-configured YAML file listing all the various
Contrail components.  This saves having to find the IP at run time and means if you have multiple instances of each component,
the script will get logs off all instances of a specified type.

An example IPs file is included in the repo.

```

python3 ./contrail_get_logs.py get_log_config.yaml analyticsdb -i unit_ips.yaml
zipping '/var/log/contrail/' up to /var/tmp/_var_log_contrail_.tgz on 172.16.0.144
grabbing /var/tmp/_var_log_contrail_.tgz
_var_log_contrail_.tgz                                                                                              100% 1865KB  78.0MB/s   00:00    
connecting to '172.16.0.144' to get container logs
getting logs for container 'analyticsdatabase_query-engine_1'
getting logs for container 'analyticsdatabase_nodemgr_1'
getting logs for container 'analyticsdatabase_cassandra_1'
getting logs for container 'analyticsdatabase_provisioner_1'
cleaning up temporary files and zipping up to 'contrail-analyticsdb-f473cba4-d015-11ea-8ad0-1ede62014782-logs.tgz'
```

### 3) Remove confidential data.
If the log files need to be shared with a 3rd party, the script can remove confidential data from the logs.
It will do the following:  
1) IPs: change the 2 most significant octets to 'X.X'  
2) MACs: change the 3 most significant octets to 'X:X:X'  
3) hostnames: read the specified string from the config file and replace with 'dummy_hostname'  
4) domain names: read the specified string from the config file and replace with 'dummy.domain.com'  

```
python3 ./contrail_get_logs.py get_log_config.yaml heat -i unit_ips.yaml -z
zipping '/var/log/heat/' up to /var/tmp/_var_log_heat_.tgz on 172.16.0.143
grabbing /var/tmp/_var_log_heat_.tgz
_var_log_heat_.tgz                                                                                                  100% 1616KB  58.6MB/s   00:00    
removing confidential data from zip file './tmp/905a2cc0-d016-11ea-8ad0-1ede62014782/X.X.0.143/var/log/heat/heat-api-cfn.log.2.gz'
removing confidential data from text file './tmp/905a2cc0-d016-11ea-8ad0-1ede62014782/X.X.0.143/var/log/heat/heat-api-cfn.log.1'
removing confidential data from zip file './tmp/905a2cc0-d016-11ea-8ad0-1ede62014782/X.X.0.143/var/log/heat/heat-engine.log.4.gz'
removing confidential data from zip file './tmp/905a2cc0-d016-11ea-8ad0-1ede62014782/X.X.0.143/var/log/heat/heat-api.log.2.gz'
removing confidential data from zip file './tmp/905a2cc0-d016-11ea-8ad0-1ede62014782/X.X.0.143/var/log/heat/heat-engine.log.3.gz'
removing confidential data from text file './tmp/905a2cc0-d016-11ea-8ad0-1ede62014782/X.X.0.143/var/log/heat/heat-engine.log.1'
removing confidential data from text file './tmp/905a2cc0-d016-11ea-8ad0-1ede62014782/X.X.0.143/var/log/heat/heat-api-cfn.log'
removing confidential data from text file './tmp/905a2cc0-d016-11ea-8ad0-1ede62014782/X.X.0.143/var/log/heat/heat-api.log.1'
removing confidential data from text file './tmp/905a2cc0-d016-11ea-8ad0-1ede62014782/X.X.0.143/var/log/heat/heat-engine.log'
removing confidential data from zip file './tmp/905a2cc0-d016-11ea-8ad0-1ede62014782/X.X.0.143/var/log/heat/heat-engine.log.2.gz'
removing confidential data from text file './tmp/905a2cc0-d016-11ea-8ad0-1ede62014782/X.X.0.143/var/log/heat/heat-api.log'
removing confidential data from zip file './tmp/905a2cc0-d016-11ea-8ad0-1ede62014782/X.X.0.143/var/log/heat/heat-api.log.3.gz'
removing confidential data from zip file './tmp/905a2cc0-d016-11ea-8ad0-1ede62014782/X.X.0.143/var/log/heat/heat-api.log.4.gz'
cleaning up temporary files and zipping up to 'heat-905a2cc0-d016-11ea-8ad0-1ede62014782-logs.tgz'
```

### 4) Help
The script uses argparse so standard help is available.

```
python3 ./contrail_get_logs.py -h
usage: contrail_get_logs.py [-h] [-d DEVICE_IP] [-i IPS_FILE] [-z]
                            [-u USERNAME]
                            config_file component

Grab contrail component logs

positional arguments:
  config_file           Location of YAML file containing log file paths and
                        strings to hide if using '-h' option
  component             Name of component: control, analytics, analyticsdb,
                        vrouter, haproxy, heat, neutron, appformix

optional arguments:
  -h, --help            show this help message and exit
  -d DEVICE_IP, --device-ip DEVICE_IP
                        get the logs from the specified IP
  -i IPS_FILE, ---ips-file IPS_FILE
                        lookup component in the IPs file and grab logs from
                        all the IPs listed
  -z, --hide-data       attempt to obfuscate things like IPs, MACs & hostnames
  -u USERNAME, --username USERNAME
                        username, default= 'ubuntu'
```