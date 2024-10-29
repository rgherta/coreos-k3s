# coreos-k3s


Install coreos

```
mkdir --parents ~/.local/share/libvirt/images/

STREAM="stable"
# as an installed binary:
coreos-installer download -s "${STREAM}" -p qemu -f qcow2.xz --decompress -C ~/.local/share/libvirt/images/

# cp /home/romh/.local/share/libvirt/images/fedora-coreos-40.20241006.3.0-qemu.x86_64.qcow2 fedora-coreos-40.qcow2


butane --pretty --strict myk3s.bu > myk3s.ign

IGNITION_CONFIG="/home/romh/Documents/coreos-k3s/myk3s.ign"
IMAGE="$PWD/fedora-coreos-40.qcow2"
VM_NAME="fcos-test-01"
VCPUS="2"
RAM_MB="2048"
STREAM="stable"
DISK_GB="20"
# For x86 / aarch64,
IGNITION_DEVICE_ARG=(--qemu-commandline="-fw_cfg name=opt/com.coreos/config,file=${IGNITION_CONFIG}")

# Setup the correct SELinux label to allow access to the config
chcon --verbose --type svirt_home_t ${IGNITION_CONFIG}

virt-install --connect="qemu:///system" --name="${VM_NAME}" --vcpus="${VCPUS}" --memory="${RAM_MB}" \
        --os-variant="fedora-coreos-$STREAM" --import --graphics=none \
        --disk="size=${DISK_GB},backing_store=${IMAGE}" \
        --network bridge=virbr0 "${IGNITION_DEVICE_ARG[@]}"
```


Install cni plugin

```
helm repo add cilium https://helm.cilium.io
helm install mycilium cilium/cilium --set ipam.operator.clusterPoolIPv4PodCIDRList="10.0.0.0/8" --set ipam.operator.clusterPoolIPv4MaskSize=24 --set kubeProxyReplacement=true --set operator.replicas=1 --set envoy.enabled=false --set k8sServicePort=6443 --set k8sServiceHost=127.0.0.1 

```