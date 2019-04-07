# ElasticProject
Configuración clúster ES 6.7 +  Kibana + Logstash

Instalación y configuración de un clúster con 3 nodos de ES (4 GB) + 1 nodo de Kibana (1 GB) + 1 nodo Logstash (2 GB)
Versión preliminar 6.7.1

Configuración de la memoria virtual adecuada para un entarno productivo
$ grep vm.max_map_count /etc/sysctl.conf
vm.max_map_count=262144

Docker ElasticSearch
docker pull docker.elastic.co/elasticsearch/elasticsearch:6.7.1

docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:6.7.1

A incluir un yaml que permita dar configuración al ESearch
-v full_path_to/custom_elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml

The container runs Elasticsearch as user elasticsearch using uid:gid 1000:1000. Bind mounted host directories and files, such as custom_elasticsearch.yml above, need to be accessible by this user. For the data and log dirs, such as /usr/share/elasticsearch/data, write access is required as well. Also see note 1 below

mkdir esdatadir
chmod g+rwx esdatadir
chgrp 1000 esdatadir

It is important to ensure increased ulimits for nofile and nproc are available for the Elasticsearch containers
--ulimit nofile=65535:65535

Swapping needs to be disabled for performance and node stability.
-e "bootstrap.memory_lock=true" --ulimit memlock=-1:-1

If you are using the devicemapper storage driver, make sure you are not using the default loop-lvm mode. Configure docker-engine to use direct-lvm instead.
Stop Docker.$ sudo systemctl stop docker
Edit /etc/docker/daemon.json. If it does not yet exist, create it. Assuming that the file was empty, add the following contents.
{
  "storage-driver": "devicemapper"
}
Start Docker.$ sudo systemctl start docker
Verify that the daemon is using the devicemapper storage driver. Use the docker info command and look for Storage Driver.
$ docker info
  Containers: 0
    Running: 0
    Paused: 0
    Stopped: 0
  Images: 0
  Server Version: 17.03.1-ce
  Storage Driver: devicemapper
  Pool Name: docker-202:1-8413957-pool
  Pool Blocksize: 65.54 kB
  Base Device Size: 10.74 GB
  Backing Filesystem: xfs
  Data file: /dev/loop0
  Metadata file: /dev/loop1
  Data Space Used: 11.8 MB
  Data Space Total: 107.4 GB
  Data Space Available: 7.44 GB
  Metadata Space Used: 581.6 KB
  Metadata Space Total: 2.147 GB
  Metadata Space Available: 2.147 GB
  Thin Pool Minimum Free Space: 10.74 GB
  Udev Sync Supported: true
  Deferred Removal Enabled: false
  Deferred Deletion Enabled: false
  Deferred Deleted Device Count: 0
  Data loop file: /var/lib/docker/devicemapper/data
  Metadata loop file: /var/lib/docker/devicemapper/metadata
  Library Version: 1.02.135-RHEL7 (2016-11-16)
<output truncated>


Consider centralizing your logs by using a different logging driver. Also note that the default json-file logging driver is not ideally suited for production use.
To configure the Docker daemon to default to a specific logging driver, set the value of log-driver to the name of the logging driver in the daemon.json file, which is located in /etc/docker/ on Linux hosts or C:\ProgramData\docker\config\ on Windows server hosts. The default logging driver is json-file. The following example explicitly sets the default logging driver to syslog:
{
  "log-driver": "syslog"
}
If the logging driver has configurable options, you can set them in the daemon.json file as a JSON array with the key log-opts. The following example sets two configurable options on the json-file logging driver:
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3",
    "labels": "production_status",
    "env": "os,customer"
  }
}





KIBANA
docker pull docker.elastic.co/kibana/kibana:6.7.1

kibana.yml

version: '2'
services:
  kibana:
    image: docker.elastic.co/kibana/kibana:6.7.1
    volumes:
      - ./kibana.yml:/usr/share/kibana/config/kibana.yml
