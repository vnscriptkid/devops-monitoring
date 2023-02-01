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
