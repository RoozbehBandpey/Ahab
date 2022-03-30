


Identify the certificate file used for the kube-api server

cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep crt
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt

--tls-cert-file=/etc/kubernetes/pki/apiserver.crt


Identify the Certificate file used to authenticate kube-apiserver as a client to ETCD Server


--etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt


Identify the key used to authenticate kubeapi-server to the kubelet server

cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep .key | grep kubelet
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key

Identify the ETCD Server Certificate used to host ETCD server

cat /etc/kubernetes/manifests/etcd.yaml | grep crt            
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt <!-- Here -->
    - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
    - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt

Identify the ETCD Server CA Root Certificate used to serve ETCD Server



ETCD can have its own CA. So this may be a different CA certificate than the one used by kube-api server.

    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt

What is the Common Name (CN) configured on the Kube API Server Certificate?



OpenSSL Syntax: openssl x509 -in file-path.crt -text -noout


openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 746249913817325712 (0xa5b3657c2bf6090)
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = kubernetes
        Validity
            Not Before: Mar 30 13:29:37 2022 GMT
            Not After : Mar 30 13:29:37 2023 GMT
        Subject: CN = kube-apiserver
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
                    00:db:ba:34:73:ae:fc:27:d8:84:6e:30:a1:ee:34:
                    41:69:16:98:3c:80:16:66:29:c9:c9:59:55:d2:b7:
                    21:d3:4c:1c:02:09:dc:f8:1e:c5:85:2f:12:73:dd:
                    65:61:e8:de:ad:bf:0d:b3:79:29:7e:33:8f:ba:7d:
                    15:48:93:5c:b1:eb:df:6d:ff:bb:df:d0:61:46:e0:
                    c6:45:ac:33:51:91:a3:d3:2d:81:1c:a0:33:39:a0:
                    da:45:72:fe:15:2e:30:d9:c3:12:3a:50:f9:8b:b7:
                    17:dc:71:7e:fc:6f:c3:19:6e:cb:76:19:dc:91:ea:
                    77:62:61:35:e6:cc:0d:e9:56:19:0b:89:d5:29:7f:
                    e0:1b:1a:11:66:df:37:d1:52:d2:fb:12:78:b8:16:
                    11:19:cb:36:9e:0c:f4:44:06:b8:90:5f:a8:7a:5b:
                    78:fa:39:28:f9:16:54:c5:82:a8:db:a4:83:73:f5:
                    48:66:77:45:4e:fd:51:60:fe:d5:d3:1a:90:83:38:
                    1f:aa:79:08:b1:c6:27:f8:8b:d7:e4:2b:f6:9a:c6:
                    fd:85:6e:f1:4a:eb:d1:0a:04:64:6d:b5:59:04:0c:
                    94:9b:66:6d:68:7d:0b:f0:be:89:fa:a4:14:4b:4b:
                    1e:8c:98:ae:be:fe:fe:77:01:52:d0:5f:ee:33:f9:
                    34:65
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            X509v3 Extended Key Usage: 
                TLS Web Server Authentication
            X509v3 Authority Key Identifier: 
                keyid:96:B6:AC:08:BD:3A:5F:83:A6:B2:4A:DB:71:CE:00:B1:82:09:FF:72

            X509v3 Subject Alternative Name: 
                DNS:controlplane, DNS:kubernetes, DNS:kubernetes.default, DNS:kubernetes.default.svc, DNS:kubernetes.default.svc.cluster.local, IP Address:10.96.0.1, IP Address:10.33.121.9
    Signature Algorithm: sha256WithRSAEncryption
         8e:85:06:b9:d5:7b:ba:6c:b5:95:b6:21:54:3a:5f:bf:49:28:
         fd:90:99:85:fb:78:85:17:6f:33:ae:e7:5a:e4:ae:b2:37:55:
         9d:69:63:c7:61:fe:15:49:87:9b:62:a9:e6:fc:a2:61:44:d9:
         80:86:12:d3:90:2f:87:5f:a3:84:2e:d6:0f:ba:39:72:38:5e:
         a6:ef:d9:5c:53:ef:21:a1:7e:9c:bf:62:b3:a5:32:03:49:4f:
         23:27:7a:f2:10:c2:0f:50:04:dd:c1:7b:86:34:37:73:7a:93:
         30:7b:51:38:e1:19:31:e8:65:e8:77:7b:f2:6c:76:87:ed:a0:
         8c:e4:d0:39:68:e3:de:d9:d0:4a:9d:32:13:1a:b1:04:06:dd:
         df:4a:72:38:54:98:eb:0b:b4:5e:a7:7b:bc:be:c9:66:14:88:
         04:73:9b:12:fa:7a:25:e2:6e:63:42:56:30:13:a9:79:50:c5:
         53:19:da:cf:c9:33:0a:11:c4:4e:8d:d4:3d:b6:bd:ee:30:7c:
         0b:84:32:a3:e3:bc:44:79:ad:95:c8:92:ca:4f:49:2b:6b:4d:
         12:41:9d:df:76:5b:a6:7a:89:1b:50:62:ff:52:a7:9f:eb:2b:
         ce:5e:60:cb:1b:c9:7b:61:44:5b:0e:3e:f0:16:38:0b:a2:6b:
         ce:f1:64:d5


What is the name of the CA who issued the Kube API Server Certificate?

What is the Common Name (CN) configured on the ETCD Server certificate?

Kubectl suddenly stops responding to your commands. Check it out! Someone recently modified the /etc/kubernetes/manifests/etcd.yaml file



You are asked to investigate and fix the issue. Once you fix the issue wait for sometime for kubectl to respond. Check the logs of the ETCD container.


cat /etc/kubernetes/manifests/etcd.yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/etcd.advertise-client-urls: https://10.33.121.9:2379
  creationTimestamp: null
  labels:
    component: etcd
    tier: control-plane
  name: etcd
  namespace: kube-system
spec:
  containers:
  - command:
    - etcd
    - --advertise-client-urls=https://10.33.121.9:2379
    - --cert-file=/etc/kubernetes/pki/etcd/server-certificate.crt
    - --client-cert-auth=true
    - --data-dir=/var/lib/etcd
    - --initial-advertise-peer-urls=https://10.33.121.9:2380
    - --initial-cluster=controlplane=https://10.33.121.9:2380
    - --key-file=/etc/kubernetes/pki/etcd/server.key
    - --listen-client-urls=https://127.0.0.1:2379,https://10.33.121.9:2379
    - --listen-metrics-urls=http://127.0.0.1:2381
    - --listen-peer-urls=https://10.33.121.9:2380
    - --name=controlplane
    - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
    - --peer-client-cert-auth=true
    - --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
    - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    - --snapshot-count=10000
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    image: k8s.gcr.io/etcd:3.4.13-0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /health
        port: 2381
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: etcd
    resources:
      requests:
        cpu: 100m
        ephemeral-storage: 100Mi
        memory: 100Mi
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 127.0.0.1
        path: /health
        port: 2381
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /var/lib/etcd
      name: etcd-data
    - mountPath: /etc/kubernetes/pki/etcd
      name: etcd-certs
  hostNetwork: true
  priorityClassName: system-node-critical
  volumes:
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
      path: /var/lib/etcd
      type: DirectoryOrCreate
    name: etcd-data
status: {}



The kube-api server stopped again! Check it out. Inspect the kube-api server logs and identify the root cause and fix the issue.



Run docker ps -a command to identify the kube-api server container. Run docker logs container-id command to view the logs.

 docker ps -a | grep kube-apiserver
ce523b406c39        ca9843d3b545           "kube-apiserver --ad…"   34 seconds ago       Exited (1) 12 seconds ago                             k8s_kube-apiserver_kube-apiserver-controlplane_kube-system_a27440c26d3df8b1f610784d5be9d57e_2
f8c9a20e015e        k8s.gcr.io/pause:3.2   "/pause"                 About a minute ago   Up About a minute                                     k8s_POD_kube-apiserver-controlplane_kube-system_a27440c26d3df8b1f610784d5be9d57e_0

If we inspect the kube-apiserver container on the controlplane, we can see that it is frequently exiting.

root@controlplane:~# docker ps -a | grep kube-apiserver
8af74bd23540        ca9843d3b545           "kube-apiserver --ad…"   39 seconds ago      Exited (1) 17 seconds ago                          k8s_kube-apiserver_kube-apiserver-controlplane_kube-system_f320fbaff7813586592d245912262076_4
c9dc4df82f9d        k8s.gcr.io/pause:3.2   "/pause"                 3 minutes ago       Up 3 minutes                                       k8s_POD_kube-apiserve-controlplane_kube-system_f320fbaff7813586592d245912262076_1
root@controlplane:~# 
If we now inspect the logs of this exited container, we would see the following errors:

root@controlplane:~# docker logs 8af74bd23540  --tail=2
W0520 01:57:23.333002       1 clientconn.go:1223] grpc: addrConn.createTransport failed to connect to {https://127.0.0.1:2379  <nil> 0 <nil>}. Err :connection error: desc = "transport: authentication handshake failed: x509: certificate signed by unknown authority". Reconnecting...
Error: context deadline exceeded
root@controlplane:~# 
This indicates an issue with the ETCD CA certificate used by the kube-apiserver. Correct it to use the file /etc/kubernetes/pki/etcd/ca.crt.

Once the YAML file has been saved, wait for the kube-apiserver pod to be Ready. This can take a couple of minutes.

