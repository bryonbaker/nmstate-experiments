# Experiment to test VLANs

To reprduce the experiment:  
1. Install the NMState operator  
2. Create an NMState instance  
3. Deploy the bridge for VLANS.  
      `Networking > NodeNetworkConfigurationPolicy > Create > With YAML`  
      Paste the `br-vlans-nncp.yaml` content into the form and click `Create`.  
4. Log on to the node and check the state of the bridge:  
   `oc debug node/<nodename>`  
   `chroot /host`  
   `sh-5.1# ovs-vsctl show`  
   ```
   <snip>
       Bridge br-vlans
        Port eth1
            Interface eth1
                type: internal
    ovs_version: "3.3.2-40.el9fdp"
   ```
5. Create a new project `shakeout-app  
6. Deploy the two NetworkAttachmentDefintions (`vlan21-nad.yaml` and `vlan92-nad.yaml`)  
   `oc apply -f vlan21-nad.yaml`  
   `oc apply -f vlan92-nad.yaml`  
7. Deploy the Shakeout App  
   `oc apply -f shakeout-app-dep.yaml`  
8. Connect to the pod in a shell and try pinging a VM on either VLAN.  

## This does not work yet... Notes are:

I can ping the ip addresses on VLAN21 and VLAN92 from within the pod. But the bridge is not connecting through to the NAD because I cannot ping a VM on VLAN21 or 92 (ip addresses 10.10.21.1, 10.10.21.2, 10.10.92.1, 10.10.92.2). But these machines are on the VLAN because I can ping them from each other.  
Note: IP address convention is `10.10.<vlan-tag>.n` where `n` is the machine number.

The bridge is defined in OVS:

```
sh-5.1# ovs-vsctl show

71cd0e60-4b3c-4abb-906f-5e698160677a
    Bridge br-int
        fail_mode: secure
        datapath_type: system
        Port c540f102aad2a34
            Interface c540f102aad2a34
<snip>
        Port br-int
            Interface br-int
                type: internal
        Port patch-br-int-to-br-ex_kmarthub.bakerapps.net
            Interface patch-br-int-to-br-ex_kmarthub.bakerapps.net
                type: patch
                options: {peer=patch-br-ex_kmarthub.bakerapps.net-to-br-int}
    Bridge br-ex
        Port enp1s0f0np0
            Interface enp1s0f0np0
                type: system
        Port patch-br-ex_kmarthub.bakerapps.net-to-br-int
            Interface patch-br-ex_kmarthub.bakerapps.net-to-br-int
                type: patch
                options: {peer=patch-br-int-to-br-ex_kmarthub.bakerapps.net}
        Port br-ex
            Interface br-ex
                type: internal
    Bridge br-vlans
        Port eth1
            Interface eth1
                type: internal
    ovs_version: "3.3.2-40.el9fdp"

```

## Deployment Details

The deployed pod has the following spec:

```
kind: Pod
apiVersion: v1
metadata:
  generateName: shakeout-app-7ff856cf9f-
  annotations:
    k8s.ovn.org/pod-networks: '{"default":{"ip_addresses":["10.128.1.18/23"],"mac_address":"0a:58:0a:80:01:12","gateway_ips":["10.128.0.1"],"routes":[{"dest":"10.128.0.0/14","nextHop":"10.128.0.1"},{"dest":"172.30.0.0/16","nextHop":"10.128.0.1"},{"dest":"100.64.0.0/16","nextHop":"10.128.0.1"}],"ip_address":"10.128.1.18/23","gateway_ip":"10.128.0.1"}}'
    k8s.v1.cni.cncf.io/network-status: |-
      [{
          "name": "ovn-kubernetes",
          "interface": "eth0",
          "ips": [
              "10.128.1.18"
          ],
          "mac": "0a:58:0a:80:01:12",
          "default": true,
          "dns": {}
      },{
          "name": "vlan-tests/vlan21",
          "interface": "net1",
          "ips": [
              "10.10.21.3"
          ],
          "mac": "92:84:ba:62:38:eb",
          "dns": {}
      },{
          "name": "vlan-tests/vlan92",
          "interface": "net2",
          "ips": [
              "10.10.92.3"
          ],
          "mac": "9a:95:72:b5:3e:39",
          "dns": {}
      }]
    k8s.v1.cni.cncf.io/networks: '[{"name": "vlan21"}, {"name": "vlan92"}]'
    openshift.io/scc: restricted-v2
    seccomp.security.alpha.kubernetes.io/pod: runtime/default
  resourceVersion: '77030'
  name: shakeout-app-7ff856cf9f-vmn9k
  uid: 3dcb7839-65cd-4fbb-bf10-3b206944ab72
  creationTimestamp: '2024-10-21T06:25:48Z'
  managedFields:
    - manager: kube-controller-manager
      operation: Update
      apiVersion: v1
      time: '2024-10-21T06:25:48Z'
      fieldsType: FieldsV1
      fieldsV1:
        'f:metadata':
          'f:annotations':
            .: {}
            'f:k8s.v1.cni.cncf.io/networks': {}
          'f:generateName': {}
          'f:labels':
            .: {}
            'f:app': {}
            'f:pod-template-hash': {}
          'f:ownerReferences':
            .: {}
            'k:{"uid":"33d8ea28-a1d5-44cb-ad68-5b54dd3a85aa"}': {}
        'f:spec':
          'f:containers':
            'k:{"name":"shakeout-app"}':
              'f:image': {}
              'f:volumeMounts':
                .: {}
                'k:{"mountPath":"/data"}':
                  .: {}
                  'f:mountPath': {}
                  'f:name': {}
              'f:terminationMessagePolicy': {}
              .: {}
              'f:resources': {}
              'f:env':
                .: {}
                'k:{"name":"POD_NAME"}':
                  .: {}
                  'f:name': {}
                  'f:valueFrom':
                    .: {}
                    'f:fieldRef': {}
              'f:terminationMessagePath': {}
              'f:imagePullPolicy': {}
              'f:ports':
                .: {}
                'k:{"containerPort":9000,"protocol":"TCP"}':
                  .: {}
                  'f:containerPort': {}
                  'f:protocol': {}
              'f:name': {}
          'f:dnsPolicy': {}
          'f:enableServiceLinks': {}
          'f:restartPolicy': {}
          'f:schedulerName': {}
          'f:securityContext': {}
          'f:terminationGracePeriodSeconds': {}
          'f:volumes':
            .: {}
            'k:{"name":"shakeout-app-storage"}':
              .: {}
              'f:name': {}
              'f:persistentVolumeClaim':
                .: {}
                'f:claimName': {}
    - manager: kube-scheduler
      operation: Update
      apiVersion: v1
      time: '2024-10-21T06:25:48Z'
      fieldsType: FieldsV1
      fieldsV1:
        'f:status':
          'f:conditions':
            .: {}
            'k:{"type":"PodScheduled"}':
              .: {}
              'f:lastProbeTime': {}
              'f:lastTransitionTime': {}
              'f:message': {}
              'f:reason': {}
              'f:status': {}
              'f:type': {}
      subresource: status
    - manager: kmarthub.bakerapps.net
      operation: Update
      apiVersion: v1
      time: '2024-10-21T06:25:50Z'
      fieldsType: FieldsV1
      fieldsV1:
        'f:metadata':
          'f:annotations':
            'f:k8s.ovn.org/pod-networks': {}
      subresource: status
    - manager: multus-daemon
      operation: Update
      apiVersion: v1
      time: '2024-10-21T06:25:51Z'
      fieldsType: FieldsV1
      fieldsV1:
        'f:metadata':
          'f:annotations':
            'f:k8s.v1.cni.cncf.io/network-status': {}
      subresource: status
    - manager: kubelet
      operation: Update
      apiVersion: v1
      time: '2024-10-21T06:25:53Z'
      fieldsType: FieldsV1
      fieldsV1:
        'f:status':
          'f:conditions':
            'k:{"type":"ContainersReady"}':
              .: {}
              'f:lastProbeTime': {}
              'f:lastTransitionTime': {}
              'f:status': {}
              'f:type': {}
            'k:{"type":"Initialized"}':
              .: {}
              'f:lastProbeTime': {}
              'f:lastTransitionTime': {}
              'f:status': {}
              'f:type': {}
            'k:{"type":"PodReadyToStartContainers"}':
              .: {}
              'f:lastProbeTime': {}
              'f:lastTransitionTime': {}
              'f:status': {}
              'f:type': {}
            'k:{"type":"Ready"}':
              .: {}
              'f:lastProbeTime': {}
              'f:lastTransitionTime': {}
              'f:status': {}
              'f:type': {}
          'f:containerStatuses': {}
          'f:hostIP': {}
          'f:hostIPs': {}
          'f:phase': {}
          'f:podIP': {}
          'f:podIPs':
            .: {}
            'k:{"ip":"10.128.1.18"}':
              .: {}
              'f:ip': {}
          'f:startTime': {}
      subresource: status
  namespace: vlan-tests
  ownerReferences:
    - apiVersion: apps/v1
      kind: ReplicaSet
      name: shakeout-app-7ff856cf9f
      uid: 33d8ea28-a1d5-44cb-ad68-5b54dd3a85aa
      controller: true
      blockOwnerDeletion: true
  labels:
    app: shakeout-app
    pod-template-hash: 7ff856cf9f
spec:
  restartPolicy: Always
  serviceAccountName: default
  priority: 0
  schedulerName: default-scheduler
  enableServiceLinks: true
  terminationGracePeriodSeconds: 30
  preemptionPolicy: PreemptLowerPriority
  nodeName: kmarthub.bakerapps.net
  securityContext:
    seLinuxOptions:
      level: 's0:c27,c19'
    fsGroup: 1000740000
    seccompProfile:
      type: RuntimeDefault
  containers:
    - resources: {}
      terminationMessagePath: /dev/termination-log
      name: shakeout-app
      env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              apiVersion: v1
              fieldPath: metadata.name
      securityContext:
        capabilities:
          drop:
            - ALL
        runAsUser: 1000740000
        runAsNonRoot: true
        allowPrivilegeEscalation: false
      ports:
        - containerPort: 9000
          protocol: TCP
      imagePullPolicy: Always
      volumeMounts:
        - name: shakeout-app-storage
          mountPath: /data
        - name: kube-api-access-jwtn2
          readOnly: true
          mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      terminationMessagePolicy: File
      image: 'quay.io/bryonbaker/shakeout-app:latest'
  serviceAccount: default
  volumes:
    - name: shakeout-app-storage
      persistentVolumeClaim:
        claimName: shakeout-app-pvc
    - name: kube-api-access-jwtn2
      projected:
        sources:
          - serviceAccountToken:
              expirationSeconds: 3607
              path: token
          - configMap:
              name: kube-root-ca.crt
              items:
                - key: ca.crt
                  path: ca.crt
          - downwardAPI:
              items:
                - path: namespace
                  fieldRef:
                    apiVersion: v1
                    fieldPath: metadata.namespace
          - configMap:
              name: openshift-service-ca.crt
              items:
                - key: service-ca.crt
                  path: service-ca.crt
        defaultMode: 420
  dnsPolicy: ClusterFirst
  tolerations:
    - key: node.kubernetes.io/not-ready
      operator: Exists
      effect: NoExecute
      tolerationSeconds: 300
    - key: node.kubernetes.io/unreachable
      operator: Exists
      effect: NoExecute
      tolerationSeconds: 300
status:
  containerStatuses:
    - restartCount: 0
      started: true
      ready: true
      name: shakeout-app
      state:
        running:
          startedAt: '2024-10-21T06:25:52Z'
      imageID: 'quay.io/bryonbaker/shakeout-app@sha256:01cba2e7724b8956713bee71c2c494c75ba2cc45ce98fce551b544ba3877aa85'
      image: 'quay.io/bryonbaker/shakeout-app:latest'
      lastState: {}
      containerID: 'cri-o://1989e9679cc86b13f14ecc61d1f4897d29db7aeb0ebc168af4020d381cc1e7cd'
  qosClass: BestEffort
  hostIPs:
    - ip: 136.144.62.61
  podIPs:
    - ip: 10.128.1.18
  podIP: 10.128.1.18
  hostIP: 136.144.62.61
  startTime: '2024-10-21T06:25:50Z'
  conditions:
    - type: PodReadyToStartContainers
      status: 'True'
      lastProbeTime: null
      lastTransitionTime: '2024-10-21T06:25:53Z'
    - type: Initialized
      status: 'True'
      lastProbeTime: null
      lastTransitionTime: '2024-10-21T06:25:50Z'
    - type: Ready
      status: 'True'
      lastProbeTime: null
      lastTransitionTime: '2024-10-21T06:25:53Z'
    - type: ContainersReady
      status: 'True'
      lastProbeTime: null
      lastTransitionTime: '2024-10-21T06:25:53Z'
    - type: PodScheduled
      status: 'True'
      lastProbeTime: null
      lastTransitionTime: '2024-10-21T06:25:50Z'
  phase: Running
```