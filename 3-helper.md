```
yum install -y wget podman bash-completion vim
wget https://github.com/RedHatOfficial/ocp4-helpernode/releases/download/v2beta1/helpernodectl
chmod u+x helpernodectl
mv helpernodectl /usr/bin/
source <(helpernodectl completion bash)
```

```
vim helpernode.yaml

version: v2
arch: "x86_64"
helper:
  name: "helper"
  ipaddr: "192.168.7.77"
  networkifacename: "eth0"
dns:
  domain: "rober.lab"
  clusterid: "ocp4"
  forwarder1: "8.8.8.8"
  forwarder2: "8.8.4.4"
dhcp:
  router: "192.168.7.1"
  bcast: "192.168.7.255"
  netmask: "255.255.255.0"
  poolstart: "192.168.7.10"
  poolend: "192.168.7.30"
  ipid: "192.168.7.0"
  netmaskid: "255.255.255.0"
bootstrap:
  name: "bootstrap"
  ipaddr: "192.168.7.20"
  macaddr: "52:54:00:60:72:67"
  disk: vda
masters:
  - name: "master0"
    ipaddr: "192.168.7.21"
    macaddr: "52:54:00:e7:9d:67"
    disk: vda
  - name: "master1"
    ipaddr: "192.168.7.22"
    macaddr: "52:54:00:80:16:23"
    disk: vda
  - name: "master2"
    ipaddr: "192.168.7.23"
    macaddr: "52:54:00:d5:1c:39"
    disk: vda
workers:
  - name: "worker0"
    ipaddr: "192.168.7.11"
    macaddr: "52:54:00:f4:26:a1"
    disk: vda
  - name: "worker1"
    ipaddr: "192.168.7.12"
    macaddr: "52:54:00:82:90:00"
    disk: vda
  - name: "worker2"
    ipaddr: "192.168.7.12"
    macaddr: "52:54:00:82:90:00"
    disk: vda
```

```
helpernodectl get-clients
tar -xzf openshift-client-linux.tar.gz -C /usr/local/bin/
tar -xzf openshift-install-linux.tar.gz -C /usr/local/bin/
tar -xzf helm.tar.gz
mv linux-amd64/helm /usr/local/bin/
rm -f /usr/local/bin/README.md
ls -l /usr/local/bin/{oc,openshift-install,kubectl,helm}

mkdir -p ~/.openshift

cat ~/.openshift/pull-secret
```

```
ssh-keygen -b 2048 -t rsa -f /root/.ssh/id_rsa -q -N ""
mkdir ~/ocp4
cd ~/ocp4

cat <<EOF > install-config.yaml
apiVersion: v1
baseDomain: rober.lab
compute:
- hyperthreading: Enabled
  name: worker
  replicas: 0
controlPlane:
  hyperthreading: Enabled
  name: master
  replicas: 3
metadata:
  name: ocp4
networking:
  clusterNetworks:
  - cidr: 10.254.0.0/16
    hostPrefix: 24
  networkType: OpenShiftSDN
  serviceNetwork:
  - 172.30.0.0/16
platform:
  none: {}
pullSecret: '$(< ~/.openshift/pull-secret)'
sshKey: '$(< ~/.ssh/id_rsa.pub)'
EOF
```

```
openshift-install create manifests
sed -i 's/mastersSchedulable: true/mastersSchedulable: false/g' manifests/cluster-scheduler-02-config.yml
cat manifests/cluster-scheduler-02-config.yml
```

```
openshift-install create ignition-configs
helpernodectl copy-ign --dir ~/ocp4/
```

```
sudo echo nameserver 8.8.8.8 > /etc/resolv.conf
```

