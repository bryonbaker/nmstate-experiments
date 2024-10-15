# Documentation:

https://docs.redhat.com/en/documentation/openshift_container_platform/4.7/html/networking/kubernetes-nmstate#virt-example-ethernet-nncp_k8s_nmstate-updating-node-network-config


# Note:
You cannot delete an object by deleting a policy!!!!!!

You need to change the desired state to absent if you want to remove a bridge device:
https://docs.redhat.com/en/documentation/openshift_container_platform/4.7/html/networking/kubernetes-nmstate#virt-removing-interface-from-nodes_k8s_nmstate-updating-node-network-config
```
apiVersion: nmstate.io/v1beta1
kind: NodeNetworkConfigurationPolicy
metadata:
  name: <br1-eth1-policy> 
spec:
  nodeSelector: 
    node-role.kubernetes.io/worker: "" 
  desiredState:
    interfaces:
    - name: br1
      type: linux-bridge
      state: absent 
    - name: eth1 
      type: ethernet 
      state: up 
      ipv4:
        dhcp: true 
        enabled: true 
```

Example policy for enabling a network port:
```
apiVersion: nmstate.io/v1
kind: NodeNetworkConfigurationPolicy
metadata:
  name: ens3f1-policy
spec:
  desiredState:
    interfaces:
      - description: Configuring eth1 on node01
        ipv4:
          dhcp: true
          enabled: true
        name: ens3f1
        state: up
        type: ethernet
  nodeSelector:
    kubernetes.io/hostname: kmarthub.bakerapps.net
```


To create a bond interface for two ports (eno1 and eno2):
```
apiVersion: nmstate.io/v1
kind: NodeNetworkConfigurationPolicy
metadata:
  name: bond0-configuration
spec:
  desiredState:
    interfaces:
      - name: bond0
        type: bond
        state: up
        ipv4:
          enabled: true
          address:
            - ip: <IP_ADDRESS>
              prefix-length: <PREFIX_LENGTH>
        link-aggregation:
          mode: active-backup
          options:
            miimon: "100"
        port:
          - name: eno1
          - name: eno2
```

To create vlans that containers and VMs can access you need a bridge port and then create the vlans to the bridge. E.g.
```
apiVersion: nmstate.io/v1
kind: NodeNetworkConfigurationPolicy
metadata:
  name: bond0-bridge-multiple-vlans
spec:
  desiredState:
    interfaces:
      - name: bond0
        type: bond
        state: up
        link-aggregation:
          mode: active-backup
          options:
            miimon: "100"
        port:
          - name: eno1
          - name: eno2
      - name: br-bond0
        type: bridge
        state: up
        bridge:
          port:
            - name: bond0
      - name: br-bond0.92
        type: vlan
        state: up
        vlan:
          base-iface: br-bond0
          id: 92
        ipv4:
          enabled: true
          address:
            - ip: 192.168.92.3
              prefix-length: 24
      - name: br-bond0.21
        type: vlan
        state: up
        vlan:
          base-iface: br-bond0
          id: 21

```

Here is an ecample without the bonded interface:

```
apiVersion: nmstate.io/v1
kind: NodeNetworkConfigurationPolicy
metadata:
  name: eno1-bridge-multiple-vlans
spec:
  desiredState:
    interfaces:
      - name: eno1
        type: ethernet
        state: up
      - name: br-eno1
        type: bridge
        state: up
        bridge:
          port:
            - name: eno1
      - name: br-eno1.92
        type: vlan
        state: up
        vlan:
          base-iface: br-eno1
          id: 92
        ipv4:
          enabled: true
          address:
            - ip: 192.168.92.3
              prefix-length: 24
      - name: br-eno1.21
        type: vlan
        state: up
        vlan:
          base-iface: br-eno1
          id: 21
```
