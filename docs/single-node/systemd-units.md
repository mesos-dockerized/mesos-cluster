### Single node mesos cluster in docker containers managed by systemd

All command should be run as root:


Add zookeeper systemd unit:
```
cat <<EOF > /etc/systemd/system/zookeeper.service
[Unit]
Description=Zookeeper in docker
Requires=docker.service
After=docker.service

[Service]
ExecStart=/usr/bin/docker run \
        --name=zookeeper \
        --restart=always \
        -p 2181:2181 \
        -p 2888:2888 \
        -p 3888:3888 \
        jeygeethan/zookeeper-cluster {EXTERNAL_IP} 1
ExecStop=/usr/bin/docker stop -t 2 zookeeper
ExecStopPost=/usr/bin/docker rm -f zookeeper
Restart=always
RestartSec=10
[Install]
WantedBy=multi-user.target
EOF
```

Setup mesos-master systemd unit:
```
cat <<EOF > /etc/systemd/system/mesos-master.service
[Unit]
Description=Mesos master in docker
Requires=docker.service
After=docker.service

[Service]
ExecStart=/usr/bin/docker run \
        --name=mesos-master \
        --net=host \
        -e MESOS_ZK=zk://{EXTERNAL_IP}:2181/mesos \
        -e MESOS_QUORUM=1 \
        -e MESOS_REGISTRY=in_memory \
        -e MESOS_NO_HOSTNAME_LOOKUP=true \
        -e MESOS_IP={EXTERNAL_IP} \
        -e MESOS_LOG_DIR=/var/log/mesos \
        -e MESOS_WORK_DIR=/var/tmp/mesos \
        quay.io/mesosdockerized/mesos-master:v1.1.0
ExecStop=/usr/bin/docker stop -t 2 mesos-master
ExecStopPost=/usr/bin/docker rm -f mesos-master
Restart=always
RestartSec=10
[Install]
WantedBy=multi-user.target
EOF
```

Setup mesos-slave systemd unit:
```
cat <<EOF > /etc/systemd/system/mesos-slave.service
[Unit]
Description=Mesos slave in docker
Requires=docker.service
After=docker.service

[Service]
ExecStart=/usr/bin/docker run \
        --net=host \
        --privileged \
        --name=mesos-slave \
        -e MESOS_MASTER=zk://{EXTERNAL_IP}:2181/mesos \
        -e MESOS_SWITCH_USER=0 \
        -e MESOS_CONTAINERIZERS=mesos,docker \
        -e MESOS_LOG_DIR=/var/log/mesos \
        -e MESOS_IP={EXTERNAL_IP} \
        -e MESOS_WORK_DIR=/var/tmp/mesos \
        -e MESOS_NO_HOSTNAME_LOOKUP=true \
        -e MESOS_SYSTEMD_ENABLE_SUPPORT=false \
        -e MESOS_EXECUTOR_REGISTRATION_TIMEOUT=5mins \
        -e MESOS_DOCKER_KILL_ORPHANS=false \
        -v /var/run/docker.sock:/var/run/docker.sock \
        -v /cgroup:/cgroup \
        -v /sys:/sys \
        -v /usr/local/bin/docker:/usr/local/bin/docker \
        quay.io/mesosdockerized/mesos-master:v1.1.0
ExecStop=/usr/bin/docker stop -t 2 mesos-slave
ExecStopPost=/usr/bin/docker rm -f mesos-slave
Restart=always
RestartSec=10
[Install]
WantedBy=multi-user.target
EOF
```
Remember to change all {EXTERNAL_IP} fields!

Enable and start all services:
```
systemctl daemon-reload
systemctl enable zookeeper
systemctl start zookeeper
systemctl enable mesos-master
systemctl start mesos-master
systemctl enable mesos-slave
systemctl start mesos-slave
```

You can monitor every service via journalctl, for example mesos-slave:
```
journalctl -f -u mesos-slave
```

If you want add framework to your mesos-cluster, check [framework][mesos-cluster-repo] section.

[mesos-cluster-repo]: https://github.com/mesos-dockerized/mesos-cluster/tree/master/docs/frameworks
