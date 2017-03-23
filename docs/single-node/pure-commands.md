### Single node mesos cluster in docker containers

Remember to change all {EXTERNAL_IP} fields!

Start zookeeper container:
```
docker run -d --name=zookeeper --restart=always -p 2181:2181 -p 2888:2888 -p 3888:3888 jeygeethan/zookeeper-cluster {EXTERNAL_IP} 1
```

Run mesos-master container:
```
docker run -d --name=mesos-master --net=host -e MESOS_ZK=zk://{EXTERNAL_IP}:2181/mesos -e MESOS_QUORUM=1 -e MESOS_REGISTRY=in_memory -e MESOS_NO_HOSTNAME_LOOKUP=true -e MESOS_IP={EXTERNAL_IP} -e MESOS_LOG_DIR=/var/log/mesos -e MESOS_WORK_DIR=/var/tmp/mesos quay.io/mesosdockerized/mesos-master:v1.1.0
```

Connect mesos-slave to your cluster:
```
docker run -d --name=mesos-slave --net=host --privileged -e MESOS_MASTER=zk://{EXTERNAL_IP}:2181/mesos -e MESOS_SWITCH_USER=0 -e MESOS_CONTAINERIZERS=mesos,docker -e MESOS_LOG_DIR=/var/log/mesos -e MESOS_IP={EXTERNAL_IP} -e MESOS_WORK_DIR=/var/tmp/mesos -e MESOS_NO_HOSTNAME_LOOKUP=true -e MESOS_SYSTEMD_ENABLE_SUPPORT=false -e MESOS_EXECUTOR_REGISTRATION_TIMEOUT=5mins -e MESOS_DOCKER_KILL_ORPHANS=false -v /var/run/docker.sock:/var/run/docker.sock -v /cgroup:/cgroup -v /sys:/sys -v /usr/local/bin/docker:/usr/local/bin/docker quay.io/mesosdockerized/mesos-slave:v1.1.0
```

You can monitor every service via docker logs, for example mesos-slave:
```
docker logs -f mesos-slave
```

If you want add framework to your mesos-cluster, check [framework][mesos-cluster-repo] section.

[mesos-cluster-repo]: https://github.com/mesos-dockerized/mesos-cluster/tree/master/docs/frameworks
