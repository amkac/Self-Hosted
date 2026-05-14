# Torrent-VPN

Torrent safely over a VPN with gluetun

## Configure LXC for Docker & VPN

Find your container number, for example mine is 101.
Edit /etc/pve/lxc/101.conf and add:

```
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net dev/net none bind,create=dir
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```

Make sure you pass through the tun device (/dev/net/tun:/dev/net/tun) as shown in compose file.

## Test gluetun connectivity

```
docker exec -it container_name bash
wget -qO- https://ipinfo.io
```

Or

```
docker run --rm --network=container:gluetun alpine:3.18 sh -c "apk add wget && wget -qO- https://ipinfo.io"
```

## Links

- Gluetun wiki: https://github.com/qdm12/gluetun-wiki
