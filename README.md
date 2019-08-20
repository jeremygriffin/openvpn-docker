# Docker vpn pass through


### Build and start as a daemon the vpn-client:

```
docker build --file Dockerfile.openvpn --tag openvpn-client .
```

### Start as a daemon the vpn-client (entrypoint is openvpn) 

```
docker run -d \
    --name vpn-client \
    --cap-add=NET_ADMIN \
    --device /dev/net/tun \
    -v ~/.openvpn:/vpn \
    openvpn-client \
        --config /vpn/client.ovpn \
        --auth-user-pass /vpn/client.up
```

* _NOTE:_ -v is the volume map to where you keep your creds, maybe ~/.ssh/


### Build the basic box, it has ssh, mysql, tini, the entry point is tini:

```
docker build --file Dockerfile.basicbox --tag basicbox .
```

# Examples:

### SSH

```
docker run -it --rm \
    -v ~/.ssh:/root/.ssh \
    --net=container:vpn-client \
    basicbox \
        ssh -i /root/.ssh/TotallyRealKey.pem ubuntu@127.0.10.2
```

* _NOTE:_ -v is the volume map to where you keep your creds, maybe ~/.ssh/ (the container side needs to be absolute)
* --net is telling the box to route through the container vpn-client (we setup above as our vpn tunnel)
* basicbox is the image name
* ssh is the command to run.  
    * here /root/.ssh/TotallyRealKey.pem is a file found locally at: ~/.ssh/TotallyRealKey.pem and mapped with the volume -v flag

### MySql (Maria)

```
docker run -it --rm  \
    --net=container:vpn-client  \
    basicbox  \
        mysql QA_DB -hqadb.mariadb.com -uselectOnly -p -A
```

* --net is telling the box to route through the container vpn-client (we setup above as our vpn tunnel)
* basicbox is the image name
* mysql is the command to run, everything else is passed as params to this mysql client.

