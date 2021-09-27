```
mkdir ~/ocp4-workingdir
cd ~/ocp4-workingdir

yum install wget vim virt* virt-install libvirt* qemu-kvm tmux ibguestfs-tools bash-completion -y

systemctl restart libvirtd
systemctl enable libvirtd

wget https://raw.githubusercontent.com/RedHatOfficial/ocp4-helpernode/master/docs/examples/virt-net.xml

virsh net-define --file virt-net.xml

virsh net-autostart openshift4
virsh net-start openshift4

wget https://raw.githubusercontent.com/RedHatOfficial/ocp4-helpernode/master/docs/examples/helper-ks.cfg -O helper-ks.cfg

wget http://ftp.iij.ad.jp/pub/linux/centos-vault/7.6.1810/isos/x86_64/CentOS-7-x86_64-Minimal-1810.iso -O /var/lib/libvirt/images/CentOS-7-x86_64-Minimal-1810.iso

virt-install --name="ocp4-aHelper" --vcpus=2 --ram=4096 \
--disk path=/var/lib/libvirt/images/ocp4-aHelper.qcow2,bus=virtio,size=30 \
--os-variant centos7.0 --network network=openshift4,model=virtio \
--boot hd,menu=on --location /var/lib/libvirt/images/CentOS-7-x86_64-Minimal-1810.iso \
--initrd-inject helper-ks.cfg --extra-args "inst.ks=file:/helper-ks.cfg" --noautoconsole

virsh start ocp4-aHelper
```

```
ssh-keygen -b 2048 -t rsa -f /root/.ssh/id_rsa -q -N ""

ssh-copy-id 192.168.7.77

ssh 192.168.7.77
```

