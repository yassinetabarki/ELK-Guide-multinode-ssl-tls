# Configuration and Installion of ELK Stack (Elasticsearch, Logstash, and Kibana) on Ubuntu 16.04 
##### the purpose of this guide is to help you configure elasticsearch and secure communication between multiple node
 
![N|Solid](https://idroot.us/wp-content/uploads/2018/02/elk-stack-logo.png)


## Prerequisites

- A Linux system running Ubuntu  16.04
- Access to a terminal window/command line (Search > Terminal)
- A user account with sudo or root privileges
- Java version 8 or 11 (required for Logstash)


## Elastic stack (ELK)
| component | version | Link
| ------ | ------ | ------ |
Elasticsearch | 7.13.4 |[Version link](https://www.elastic.co/downloads/past-releases/elasticsearch-7-13-4)
| Kibana | 7.13.4|[Version link](https://www.elastic.co/downloads/past-releases/kibana-7-13-4)
| logstash | 7.13.4 | [Version link](https://www.elastic.co/downloads/past-releases/logstash-7-13-4)
| filebeat | 7.13.4 | [Version link]()
| metricbeat | 7.13.4 | [Version link]()


## Elastic stack (ELK)
| Host Name | Node Name | IP Address
| ------ | ------ |----- 
| ES1 | node-1 | 62.4.8.223
| metricbeat | node-2 | 62.4.8.221

# Download and install  Elasticsearch, Logstash, and Kibana
## intall and configure elasticsearch
download the Linux package appropriate for your Linux distribution. For example, on Ubuntu 64-bit you would get the  `.deb` package

```sh
    wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.13.4-amd64.deb
```
Install the package (as root). On Ubuntu
```sh
     dpkg -i elasticsearch-7.13.4-amd64.deb
```
the same goes with kibana and logstash
Configure Elasticsearch
1. Elasticsearch uses a configuration file to control how it behaves. Open the configuration file for editing in a text editor of your choice. We will be using nano:
 

````
    sudo nano /etc/elasticsearch/elasticsearch.yml
````
2. You should see a configuration file with several different entries and descriptions. Scroll down to find the following entries:

````sh
    network.host: 62.4.8.223
    
    http.port: 9200
````

3. Just below, find the Discovery section. We are adding one more line, as we are configuring a single node cluster:

````sh
discovery.type: single-node
````
### Start Elasticsearch
1. Start the Elasticsearch service by running a systemctl command:

````sh
sudo systemctl start elasticsearch.service
````

It may take some time for the system to start the service. There will be no output if successful.

2. Enable Elasticsearch to start on boot:

````sh
sudo systemctl enable elasticsearch.service
````
Test Elasticsearch
````sh
curl -X GET "62.4.8.223:9200"
````
## intall and configure kibana
It is recommended to install Kibana next. Kibana is a graphical user interface for parsing and interpreting collected log files.

1. Run the following command to install Kibana:

````sh
cd /usr/share

wget https://artifacts.elastic.co/downloads/kibana/kibana-7.13.4-amd64.deb
````
Install the package (as root). On Ubuntu under  
```sh
     dpkg -i kibana-7.13.4-amd64.deb
```
2. Allow the process to finish. Once finished, it’s time to configure Kibana.

Configure Kibana

1. Next, open the kibana.yml configuration file for editing:

````sh
sudo nano /etc/kibana/kibana.yml
````

2. Delete the # sign at the beginning of the following lines to activate them:


````sh
server.port: 5601
````

````sh
server.host: "62.4.8.223"
````

````sh
server.name: "MYELK"

elasticsearch.hosts: ["http://62.4.8.223:9200"]
````

#### 1. Start the Kibana service:


````sh
sudo systemctl start kibana
````
There is no output if the service starts successfully.

#### 2. Next, configure Kibana to launch at boot:

````sh
sudo systemctl enable kibana


sudo ufw allow 5601/tcp
````

#### install and configure logstash 
````sh
wget https://artifacts.elastic.co/downloads/logstash/logstash-7.13.4-amd64.deb
````

Install the package (as root). On Ubuntu under  

```sh
     dpkg -i logstash-7.13.4-amd64.deb
```

Start and Enable Logstash
1. Start the Logstash service:
```sh
sudo systemctl start logstash
```
2. Enable the Logstash service:
```sh
sudo systemctl enable logstash
```
3. To check the status of the service, run the following command:
```sh
sudo systemctl status logstash
```


# ELASTICSEARCH CLUSTER Multi nodes configuration
| Hostname | Node Name | IP Address
| ------ | ------ | ------ |
ES1 | node-1 | 62.4.8.223
ES2 | node-2 | 62.4.8.221

> After installing elasticsearch you need to configure
  
```sh
sudo nano /etc/elasticsearch/elasticsearch.yml:
```

## Node 1

```sh
cluster.name: MYELK
node.name: node-1
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
network.host: ["localhost","62.4.8.223"]
discovery.seed_hosts: ["62.4.8.223"]
cluster.initial_master_nodes: ["node-1"]

```

## Node 2

```sh
cluster.name: MYELK
node.name: node-1
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
network.host: ["localhost","62.4.8.221"]
discovery.seed_hosts: ["62.4.8.223"]
cluster.initial_master_nodes: ["node-1"]

```

# ES cluster configuration - HTTPS and TLS Security

- Preparations
- Create SSL certificates and enable TLS for Elasticsearch on node1
- Enable TLS for Kibana on node1
- Enable TLS for Elasticsearch on node2
- Prepare Logstash users on node1
- Enable TLS for Logstash on node1
- Run Filebeat and set up TLS on node1
- Use Filebeat to ingest data

# Configure /etc/hosts file
# /etc/hosts file for [node1]
127.0.0.1 kibana.local logstash.local
62.4.8.223 node-1
62.4.8.221 node-2

# /etc/hosts file for [node2] 
62.4.8.223 node-1
62.4.8.221 node-2

# Step 2. Create SSL certificates and enable TLS for Elasticsearch on node1

```
[root@node1 ~]# ES_HOME=/usr/share/elasticsearch
[root@node1 ~]# ES_PATH_CONF=/etc/elasticsearch
```

# [2-2] Create tmp folder
```
[root@node1 ~]# mkdir tmp
[root@node1 ~]# cd tmp/
[root@node1 tmp]# mkdir cert_blog
```

# [2-3] Create instance yaml file
```sh
vi ~/tmp/cert_blog/instance.yml 
```
# add the instance information to yml file

```sh

instances:
  - name: "node-1"
    ip:
      - "62.4.8.223"

  - name: "node-2"
    ip:
      - "62.4.8.221"
  - name: "my-kibana"
    ip:
      - "62.4.8.223"

  - name: "logstash"
    ip:
      - "62.4.8.223"
```
# [2-4] Generate CA and server certificates (once Elasticsearch is installed)
```sh
cd $ES_HOME
bin/elasticsearch-certutil cert --keep-ca-key --pem --in ~/tmp/cert_blog/instance.yml --out ~/tmp/cert_blog/certs.zip
```
# [2-5] Unzip the certificates
```sh
cd ~/tmp/cert_blog
unzip certs.zip -d ./certs
```

# [2-6] Elasticsearch TLS setup
#### [2-6-1] Copy cert file to config folder
```sh
[root@node1 ~]# cd $ES_PATH_CONF
[root@node1 elasticsearch]# pwd
/etc/elasticsearch
[root@node1 elasticsearch]# mkdir certs
[root@node1 elasticsearch]# cp ~/tmp/cert_blog/certs/ca/ca* ~/tmp/cert_blog/certs/node1/* certs

[root@node1 elasticsearch]# ll certs
total 12
-rw-r--r--. 1 root elasticsearch 1834 Apr 12 08:47 ca.crt
-rw-r--r--. 1 root elasticsearch 1834 Apr 12 08:47 ca.key
-rw-r--r--. 1 root elasticsearch 1509 Apr 12 08:47 node1.crt
-rw-r--r--. 1 root elasticsearch 1679 Apr 12 08:47 node1.key
[root@node1 elasticsearch]#
```

# [2-6-2] Configure elasticsearch.yml
```sh
[root@node1 elasticsearch]# vi elasticsearch.yml 
```

## add the following contents
```sh
xpack.security.enabled: true
xpack.security.http.ssl.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.http.ssl.key: certs/node-1.key
xpack.security.http.ssl.certificate: certs/node-1.crt
xpack.security.http.ssl.certificate_authorities: certs/ca.crt
xpack.security.transport.ssl.key: certs/node-1.key
xpack.security.transport.ssl.certificate: certs/node-1.crt
xpack.security.transport.ssl.certificate_authorities: certs/ca.crt
```

## [2-6-3] Start and check cluster log
```sh
grep '\[node1\] started' /var/log/elasticsearch/elasticsearch.log 
[o.e.n.Node               ] [node1] started
```
##  [2-6-4] Set built-in user password
```sh
cd $ES_HOME
bin/elasticsearch-setup-passwords auto -u 
```
# Step 3. Enable TLS for Kibana on node1
 - Adapt these variable paths depending on where and how Kibana was downloaded:
```sh
 [root@node1 ~]# KIBANA_HOME=/usr/share/kibana
 [root@node1 ~]# KIBANA_PATH_CONFIG=/etc/kibana
```
# [3-2] Create config and config/certs folder and copy certs (once Kibana is installed)
Copy the certification files created previously in step 2-4 and paste on kibana/config/certs.
```sh
[root@node1 kibana]# ls config/certs
total 12
ca.crt
my-kibana.crt
my-kibana.key
```

# [3-3] Configure kibana.yml
 Remember to use the password generated for the built-in user above. You need to replace <kibana_password> with the password that was defined in step 2-6-4.
 ```sh
 [root@node1 kibana]# vi kibana.yml 
server.name: "my-kibana"
server.host: "kibana.local"
server.ssl.enabled: true
server.ssl.certificate: /etc/kibana/config/certs/my-kibana.crt
server.ssl.key: /etc/kibana/config/certs/my-kibana.key
elasticsearch.hosts: ["https://node1.elastic.test.com:9200"]
elasticsearch.username: "kibana"
elasticsearch.password: "<kibana_password>"
elasticsearch.ssl.certificateAuthorities: [ "/etc/kibana/config/certs/ca.crt" ]
```

# Step 4. Enable TLS for Elasticsearch on node2

## [4-1] Set environment variables
```sh
[root@node2 ~]# ES_HOME=/usr/share/elasticsearch
[root@node2 ~]# ES_PATH_CONF=/etc/elasticsearch
```

## [4-2] Set up TLS on node2
You can use the scp command to copy certificates from node1 to node2. Both nodes require the certificate and key in order to secure the connection. In a Production environment, it is recommended to use a properly signed key for each node. For demonstration purposes, we are using an automatically generated CA certificate and multi-DNS hostname certificate signed by our generated CA.

```
[root@node2 ~]# cd $ES_PATH_CONF
[root@node2 elasticsearch]# pwd
/etc/elasticsearch
[root@node2 elasticsearch]# mkdir certs
[root@node2 elasticsearch]# cp ~/tmp/cert_blog/certs/ca/ca.crt ~/tmp/cert_blog/certs/node2/* certs  
[root@node2 elasticsearch]# 
[root@node2 elasticsearch]# ll certs
total 12
-rw-r--r--. 1 root elasticsearch 1834 Apr 12 10:55 ca.crt
-rw-r--r--. 1 root elasticsearch 1509 Apr 12 10:55 node2.crt
-rw-r--r--. 1 root elasticsearch 1675 Apr 12 10:55 node2.key
```

## [4-3] Configure elasticsearch.yml

```sh
[root@node2 elasticsearch]# vi elasticsearch.yml 
node.name: node2
network.host: node2.elastic.test.com
xpack.security.enabled: true
xpack.security.http.ssl.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.http.ssl.key: certs/node2.key
xpack.security.http.ssl.certificate: certs/node2.crt
xpack.security.http.ssl.certificate_authorities: certs/ca.crt
xpack.security.transport.ssl.key: certs/node2.key
xpack.security.transport.ssl.certificate: certs/node2.crt
xpack.security.transport.ssl.certificate_authorities: certs/ca.crt
discovery.seed_hosts: [ "62.4.8.223" ]
```
