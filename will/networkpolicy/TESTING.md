## MultiNetworkPolicy

### Block traffic on secondary network in 2001-dmz

```
apiVersion: k8s.cni.cncf.io/v1beta1
kind: MultiNetworkPolicy
metadata:
  annotations:
    k8s.v1.cni.cncf.io/policy-for: 2001-dmz-br-vlans
  name: deny-all-ingress-secondary-2001-dmz
  namespace: 2001-dmz
spec:
  ingress: []
  podSelector: {}
  policyTypes:
    - Ingress
```

```
will@wcushen-mac multinetworkpolicy % oc apply -f multinetpol-deny-all-ingress-secondary-2001-dmz.yaml
multinetworkpolicy.k8s.cni.cncf.io/deny-all-ingress-secondary-2001-dmz created
```

--- Pod to VM (static) on 10.0.201.0/24 subnet

```
will@wcushen-mac networkpolicy % oc rsh pod-additional-net-static-658bb46df-cbqtw
sh-5.1# ping 10.0.201.100
PING 10.0.201.100 (10.0.201.100) 56(84) bytes of data.
^C
```

## BaselineAdminNetworkPolicy

### Block any dmz VM traffic to other workloads in 2001-dmz

```
ingress:
    - action: Deny
      from:
        - pods:
            namespaceSelector:
              matchLabels:
                zone: dmz
            podSelector:
              matchExpressions:
                - key: kubevirt.io
                  operator: In
                  values:
                    - virt-launcher
```

```
will@wcushen-mac baselineadminnetworkpolicy % oc apply -f netpol-banp-default.yaml
baselineadminnetworkpolicy.policy.networking.k8s.io/default created
```

--- Curl dmz welcome pod from dmz VM

```
will@wcushen-mac networkpolicy % oc rsh virt-launcher-fedora-dhcp-qhkws
sh-5.1$ curl 10.128.1.241:8080
^C
```

--- OVERRIDE with standard NetPol

```
will@wcushen-mac namespacenetworkpolicy % oc apply -f netpol-allow-ingress-from-all-dmz.yaml
networkpolicy.networking.k8s.io/allow-ingress-from-all-dmz-ns created
```

--- Re-try curl

```
sh-5.1$ curl 10.128.1.241:8080
Hello World ! Welcome to OpenShift from welcome-5bcd78bf67-8jmg5:10.128.1.241
```

