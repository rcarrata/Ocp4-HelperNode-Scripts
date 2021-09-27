#!/bin/bash

Changing VMs boot order to boot from network and start them"

```
export vms="ocp4-bootstrap ocp4-master0 ocp4-master1 ocp4-master2 ocp4-worker0 ocp4-worker1 ocp4-worker2"

for vmname in $vms
do
  virsh destroy $vmname >/dev/null 2>&1
  virsh dumpxml $vmname > $vmname.xml
  sed -i "s#boot dev.*#boot dev\=\'network\'\/>#g" $vmname.xml
  virsh define --file $vmname.xml >/dev/null 2>&1
  virsh start  $vmname >/dev/null 2>&1
done
```

Waiting to finish PXE boot and changing VMs boot order to boot from hd

```
numfinished=0
numvms=0
for i in $vms; do numvms=$((numvms + 1)) ; done

while [ $numfinished != $numvms ]
do
  for vmname in $vms
  do
    lastentry=$(tail -n 1 /var/log/libvirt/qemu/$vmname.log)
    if [ "$(echo $lastentry | awk -F : '{print $1}')" == "inputs_channel_detach_tablet" ]; then
      virsh destroy $vmname >/dev/null 2>&1
      numfinished=$((numfinished + 1))
    fi
  done
done

sleep 5

for vmname in $vms
do
  sed -i "s#boot dev.*#boot dev\=\'hd\'\/>#g" $vmname.xml
  virsh define --file $vmname.xml >/dev/null 2>&1
  virsh start  $vmname >/dev/null 2>&1
  rm -rf $vmname.xml
done
```

```
openshift-install wait-for bootstrap-complete --log-level debug
```

```
export KUBECONFIG=/root/ocp4/auth/kubeconfig

oc get csr -o go-template='{{range .items}}{{if not .status}}{{.metadata.name}}{{"\n"}}{{end}}{{end}}' | xargs oc adm certificate approve

oc patch configs.imageregistry.operator.openshift.io cluster --type merge --patch '{"spec":{"managementState":"Managed"}}'
oc patch configs.imageregistry.operator.openshift.io cluster --type merge --patch '{"spec":{"storage":{"emptyDir":{}}}}'
oc patch configs.imageregistry.operator.openshift.io/cluster --type merge -p '{"spec":{"defaultRoute":true}}'
```

```
openshift-install wait-for install-complete
```

```
mkdir .kube
cp -pr ocp4/auth/kubeconfig .kube/config

oc completion bash > oc_bash_completion
sudo cp oc_bash_completion /etc/bash_completion.d/
```
