## download

```bash
cd ~/software
wget https://downloads.apache.org/zookeeper/zookeeper-3.6.3/apache-zookeeper-3.6.3-bin.tar.gz
tar -xzf apache-zookeeper-3.6.3-bin.tar.gz
```

## config

```bash
cd apache-zookeeper-3.6.3-bin
cp conf/zoo_sample.cfg conf/zoo.cfg
```

## start

```bash
./bin/zkServer.sh start
```
