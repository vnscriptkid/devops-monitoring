## Setup prometheus

Now that we have what we're monitoring set up, we need to get our monitoring tool itself up and running, complete with a service file. 
Prometheus is a pull-based monitoring system that scrapes various metrics set up across our system and stores them in a time-series database, 
where we can use a web UI and the PromQL language to view trends in our data. 
Prometheus provides its own web UI, but we'll also be pairing it with Grafana later, as well as an alerting system.

1. Create a system user for Prometheus:
```console
sudo useradd --no-create-home --shell /bin/false prometheus
```
* This command creates a new user on a Linux system named "prometheus"
* The "--no-create-home" option ensures that no home directory is created for the user
* The "--shell /bin/false" option sets the user's shell to "/bin/false", which is a special shell used for system accounts that provides no interactive login capability.

2. Create the directories in which we'll be storing our configuration files and libraries:
```console
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
```
* `/etc` is the configuration directory and it contains configuration files for various applications and services on the system
* `/var/lib` is the data directory and it contains data files and databases used by various applications and services on the system
 
3. Set the ownership of the `/var/lib/prometheus` directory:
```console
sudo chown prometheus:prometheus /var/lib/prometheus
```
* Changes the ownership of the directory "/var/lib/prometheus" to the user "prometheus" and the group "prometheus"

4. Pull down the `tar.gz` file from the [Prometheus downloads page](https://prometheus.io/download/):
```console
cd /tmp/
wget https://github.com/prometheus/prometheus/releases/download/v2.7.1/prometheus-2.7.1.linux-amd64.tar.gz
```
  
5. Extract the files:
```console
tar -xvf prometheus-2.7.1.linux-amd64.tar.gz
```
* x: extract (uncompress and extract the contents of the archive)
* v: verbose (show detailed information about the extraction process)
* f: file (specifies the archive file name that is to be extracted)
                                                                     
6. Move the configuration file and set the owner to the `prometheus` user:
```console
cd prometheus-2.7.1.linux-amd64
sudo mv console* /etc/prometheus
sudo mv prometheus.yml /etc/prometheus
sudo chown -R prometheus:prometheus /etc/prometheus
```
* The option -R (or --recursive) is used to apply the change recursively to all subdirectories and files within the target directory
                                                               
7. Move the binaries and set the owner:
```console
sudo mv prometheus /usr/local/bin/
sudo mv promtool /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
```
* `/usr/local/bin` is used to store executables for locally installed software, typically included in the system's PATH environment
                                                                     
8. Create the service file:
```console
sudo vim /etc/systemd/system/prometheus.service
```

* Add the following to the file:

```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```
    
* Save and exit. (For VIM, press `ESC`, `:`, `wq` to save and exit.)

 9. Reload systemd:
```console
sudo systemctl daemon-reload
```
* Reloads the system manager configuration, which allows any changes made to the system manager configuration files to take effect.
* Used when you have made changes to the Systemd configuration files

 10. Start Prometheus, and make sure it automatically starts on boot:
```console
sudo systemctl start prometheus
sudo systemctl enable prometheus
```
    
11. Visit Prometheus in your web browser at <PUBLICIP>:9090.
 
## Setup Alert Manager
 
1. Create the alertmanager system user:
```console
sudo useradd --no-create-home --shell /bin/false alertmanager
```
 
2. Create the /etc/alertmanager directory:
```console
sudo mkdir /etc/alertmanager
```
 
3. Download Alertmanager from the Prometheus downloads page (https://prometheus.io/download/):
```console
cd /tmp/
wget https://github.com/prometheus/alertmanager/releases/download/v0.16.1/alertmanager-0.16.1.linux-amd64.tar.gz
tar -xvf alertmanager-0.16.1.linux-amd64.tar.gz
```

4. Move the binaries:
```console
cd alertmanager-0.16.1.linux-amd64
sudo mv alertmanager /usr/local/bin/
sudo mv amtool /usr/local/bin/
``` 
 
5. Set the ownership of the binaries:
```console
sudo chown alertmanager:alertmanager /usr/local/bin/alertmanager
sudo chown alertmanager:alertmanager /usr/local/bin/amtool
```
 
6. Move the configuration file into the /etc/alertmanager directory:
```console
sudo mv alertmanager.yml /etc/alertmanager/
```
 
7. Set the ownership of the /etc/alertmanager directory:
```console
sudo chown -R alertmanager:alertmanager /etc/alertmanager/
```
 
8. Create the alertmanager.service file for systemd:
```console
sudo $EDITOR /etc/systemd/system/alertmanager.service
```
Copy and Paste content:
```
[Unit]
Description=Alertmanager
Wants=network-online.target
After=network-online.target

[Service]
User=alertmanager
Group=alertmanager
Type=simple
WorkingDirectory=/etc/alertmanager/
ExecStart=/usr/local/bin/alertmanager \
    --config.file=/etc/alertmanager/alertmanager.yml
[Install]
WantedBy=multi-user.target
```
Save and exit.

9. Stop Prometheus, and then update the Prometheus configuration file to use Alertmanager:
```console
sudo systemctl stop prometheus
sudo $EDITOR /etc/prometheus/prometheus.yml
```

```
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - localhost:9093
```
 
10. Reload systemd, and then start the prometheus and alertmanager services:
```console
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl start alertmanager
sudo systemctl enable alertmanager
```

Visit PUBLICIP:9093 in your browser to confirm Alertmanager is working.
 
## Setup Grafana

1. Install the prerequisite package:
```console
sudo apt-get install libfontconfig
```

2. Download and install Grafana using the .deb package provided on the Grafana download page (https://grafana.com/grafana/download):
```console
wget https://dl.grafana.com/oss/release/grafana_5.4.3_amd64.deb
sudo dpkg -i grafana_5.4.3_amd64.deb
```
* The `dpkg` command is a package manager for Debian-based systems
* `-i` option tells it to install the package specified by the file name that follows
 
3. Ensure Grafana starts at boot:
```console
sudo systemctl enable --now grafana-server
```
* The --now option of the systemctl enable command is equivalent to running `sudo systemctl start grafana-server`
 
4. Access Grafana's web UI by going to IPADDRESS:3000.

5. Log in with the username admin and the password admin. Reset the password when prompted.

6. Add a Data Source
* Click Add data source on the homepage.
* Select Prometheus.
* Set the URL to http://localhost:9090.
* Click Save & Test.

7. Add a Dashboard
* From the left menu, return Home.
* Click New dashboard. The dashboard is automatically created.
* Click on the gear icon to the upper right.
* Set the Name of the dashboard to Forethought.
* Save the changes.

## Setup Node Exporter

1. Create a system user:

```console
sudo useradd --no-create-home --shell /bin/false node_exporter
```
 
2. Download the Node Exporter from Prometheus's download page (http://prometheus.io/download):
```console
cd /tmp/

wget https://github.com/prometheus/node_exporter/releases/download/v0.17.0/node_exporter-0.17.0.linux-amd64.tar.gz

tar -xvf node_exporter-0.17.0.linux-amd64.tar.gz
```
 
3. Move into the newly created directory, Move the provided binary:

```console
cd node_exporter-0.17.0.linux-amd64/

sudo mv node_exporter /usr/local/bin/
```
 
4. Set the ownership:
```console
$ sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

5. Create a systemd service file:
```console
sudo vim /etc/systemd/system/node_exporter.service
```

```
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```
Save and exit when done.

6. Start the Node Exporter:
```console
sudo systemctl daemon-reload
sudo systemctl start node_exporter
```
 
7. Add the endpoint to the Prometheus configuration file:

```console
sudo $EDITOR /etc/prometheus/prometheus.yml
```

```
- job_name: 'nodeexporter'
  static_configs:
  - targets: ['localhost:9100']
```

8. Restart Prometheus:
```console
sudo systemctl restart prometheus
```
 
9. Navigate to the Prometheus web UI. Using the expression editor, search for cpu, meminfo, and related system terms to view the newly added metrics.

10. Search for `node_memory_MemFree_bytes` in the expression editor; shorten the time span for the graph to be about 30 minutes of data.

11. Back on the terminal, download and run stress to cause some memory spikes:
```console
sudo apt-get install stress
stress -m 2
```
- The `-m 2` option specifies the number of worker threads that will be used to put stress on the system's memory
 - allocate a large amount of memory
 - perform various memory-intensive operations, such as read/write, allocate/deallocate memory
 
Wait for about one minute, and then view the graph to see the difference in activity.

References
Node Exporter Metrics
https://github.com/prometheus/node_exporter/blob/master/README.md
 
 
 
