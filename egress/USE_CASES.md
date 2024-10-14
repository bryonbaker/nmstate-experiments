# VMs

## Without EgressIP / MetalLB 

[fedora@fedora-static ~]$ python -m http.server 9000
Serving HTTP on 0.0.0.0 port 9000 (http://0.0.0.0:9000/) ...
10.0.203.1 - - [23/Jul/2024 10:57:09] "GET / HTTP/1.1" 200 -


[will@dl380-02 ~]$ curl 10.0.203.100:9000
[fedora@fedora-static ~]$ python -m http.server 9000
Serving HTTP on 0.0.0.0 port 9000 (http://0.0.0.0:9000/) ...
10.0.203.1 - - [23/Jul/2024 11:01:41] "GET / HTTP/1.1" 200 -

## EgressIP applied 


[fedora@fedora-static ~]$ python -m http.server 9000
Serving HTTP on 0.0.0.0 port 9000 (http://0.0.0.0:9000/) ...

will@wcushen-mac /tmp % oc rsh welcome-5bcd78bf67-57xmp
sh-5.1$ curl 10.0.203.100:9000

