## HTTP接口

### `GET /v1/version`

```
curl http://192.168.1.61:8123/v1/version
{ 
    "Service": "Mesos-DNS", 
    "URL": "https://github.com/mesosphere/mesos-dns", "Version": "v0.6.0" 
}
```

### GET \/v1\/config

```
curl http://192.168.1.61:8123/v1/config
{ 
    "RefreshSeconds": 30, 
    "Port": 61053, 
    "Timeout": 5, 
    "StateTimeoutSeconds": 300, 
    "ZkDetectionTimeout": 30, 
    "HttpPort": 8123, 
    "TTL": 60, "SOAMname": "ns1.mesos.", "SOARname": "root.ns1.mesos.", 
    "Masters": null, 
    "Resolvers": [ "8.8.4.4", ], 
    ...... 
}
```

### GET \/v1\/hosts\/{host}

```
curl http://192.168.1.61:8123/v1/hosts/nirvana.marathon.mesos
[ 
    { "host": "nirvana.marathon.mesos.", "ip": "192.168.1.73" } 
]
```

### GET \/v1\/services\/{service}

```
curl http://192.168.1.61:8123/v1/services/_nirvana._tcp.marathon.mesos
[ 
    { 
        "service": "_nirvana._tcp.marathon.mesos", 
        "host": "nirvana-qmxxu-s4.marathon.mesos.", 
        "ip": "192.168.1.73", "port": "27437" 
    } 
]
```

