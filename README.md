# vpn-socks5-emule

Kubernetes project based on ROOK ceph distribuited storage and NFS share. 
It permits to use a vpn connection to shadow all torrent/emule activities by a SOCKS5 server (as a proxy for your torrent client) or directly inside the emule container thanks to the webgui admin page.
There are 4 containers inside the created POD: openvpn-client, socks5 server, emule client, busybox container to add the route to the internal DNS K8s server (coreDNS)

>Openvpn-client: based on the project of [chuckremes/openvpn-client](https://hub.docker.com/r/chuckremes/openvpn-client/).  The deployment creates a vpn connection to the VyprVPN provider at UK location.  Customizable with env variables. Take a look to the original project to change the parameters according to your VPN provider

>Socks5 server: based on the project of [xkuma/socks5](https://hub.docker.com/r/xkuma/socks5/). It creates a SOCKS5 proxy server listening on 1080 (customizable)

>Emule: based on the project of [reimashi/emule](https://hub.docker.com/r/reimashi/emule). It creates an emule container listening on 4711

>"Add-route" busybox: controller container to add the route to K8s internal dns server since the default gateway is being deleted in the shared pod namespace by openvpn-client. Change the internal dns server according to your k8s setup

The **namespace.yaml** creates the "vpn" namespace. Inside it all the K8s components will be installed (services, pod, pv, pvc, secret). If you want to rename it, remember to change the label in all the following yaml

The authentication to the VPN provider is configured in **secrets.yaml** on base64 translation.
Put your own username/password converted in base64 format

`echo "my_login_username_2VPNprovider" | base64  --> bXlfbG9naW5fdXNlcm5hbWVfMlZQTnByb3ZpZGVyCg==`

`echo "my_login_password_2VPNprovider" | base64  --> bXlfbG9naW5fcGFzc3dvcmRfMlZQTnByb3ZpZGVyCg==`

The **service.yaml** configures the LoadBalancer (in my case [MetalLB](https://metallb.universe.tf/) to accept connections to the listening ports of the containers.  In order to share more ports on the same ip address a metallb.universe.tf/allow-shared-ip parameter has been used. Change it according to your infrastructure

The **pvc.yaml** creates the PV for the emule temporary downloads using the rook storage class.  Change it according to your storageclass k8s installation.  The 50Gi size is purely indicative

The **deployment.yaml** creates the pod with the 4 containers inside: openvpn-client, socks5, add-route, emule.  They share the same docker pod namespace hence they see the same network and volumes. This configuration permits to route all the traffic inside the vpn tunnel

Pay attention to the add-route container deployment.  Change the IP in the following command:

`"while true; do (route -n | grep` **10.43.0.10**`); if [[ $? -eq 1 ]]; then route add -net` **10.43.0.10** `netmask 255.255.255.255 gw 169.254.1.1; fi; sleep 600; done;"`

according to your K8s internal dns address

## Version Guidance

| Name | Version | 
|---------------------|---------|
| Kubernetes   | v1.14.8        | 
| Rook         | v0.9           |