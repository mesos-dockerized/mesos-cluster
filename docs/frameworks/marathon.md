## Attaching marathon framework to your cluster

### Pure docker command

```
docker run -d --name=marathon --net=host -e MARATHON_HTTP_ADDRESS={NODE_IP} -e MARATHON_MASTER=zk://{ZOOKEEPER_IP}:2181/mesos -e MARATHON_ZK=zk://{ZOOKEEPER_IP}:2181/marathon -e LIBPROCESS_IP={NODE_IP} mesoscloud/marathon:1.1.1-centos-7
```

### Systemd solution
```
cat <<EOF > /etc/systemd/system/marathon.service
[Unit]
Description=Marathon scheduler for mesos in docker
Requires=docker.service
After=docker.service

[Service]
ExecStart=/usr/bin/docker run \
        --name=marathon \
        --net=host \
        -e MARATHON_HTTP_ADDRESS={NODE_IP} \
        -e MARATHON_MASTER=zk://{ZOOKEEPER_IP}:2181/mesos \
        -e MARATHON_ZK=zk://{ZOOKEEPER_IP}:2181/marathon \
        -e LIBPROCESS_IP={NODE_IP} \
        mesoscloud/marathon:1.1.1-centos-7
ExecStop=/usr/bin/docker stop -t 2 marathon
ExecStopPost=/usr/bin/docker rm -f marathon
Restart=always
RestartSec=10
[Install]
WantedBy=multi-user.target
EOF
```

Enable and start all services:
```
systemctl daemon-reload
systemctl enable marathon
systemctl start marathon
```
