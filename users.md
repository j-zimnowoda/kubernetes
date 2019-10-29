# Create a new user that uses certificates to authenticates with cluster.
A user should have permission to all namespaces and to get/list/watch available pods in all namespaces namespaces


Generate private key:
```
$ openssl genrsa -out myuser.key 2048
```

Generate CSR:
```
$ openssl req -new -key myuser.key -out myuser.csr -subj "/CN=myuser/O=myusergroup" 
```

Verify if CSR containes proper Common Name and Organization fields
```
$ openssl req -in myuser.csr -noout -text
Certificate Request:
    Data:
        Version: 0 (0x0)
        Subject: CN=myuser, O=myusergroup

```

Sign CSR:
```
$ sudo openssl x509 -req \
-in myuser.csr \
-out myuser.crt \
-CA /etc/kubernetes/pki/ca.crt \
-CAkey /etc/kubernetes/pki/ca.key \
-CAcreateserial \
-days 365
```

Verify certificate data
```
$ openssl x509 -in myuser.crt -noout -text
Certificate:
    Data:
        Version: 1 (0x0)
        Serial Number: 15900724891612191733 (0xdcaab90b13c19bf5)
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN=kubernetes
        Validity
            Not Before: Oct 29 16:56:35 2019 GMT
            Not After : Oct 28 16:56:35 2020 GMT
        Subject: CN=myuser, O=myusergroup
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
```

Store credientials in ~/.kube/config 
```
$ k config set-credentials myuser --client-certificate=myuser.crt --client-key=myuser.key 
```

Store context in ~/.kube/config 
```
k config set-context myuser-context --cluster=kubernetes --user=myuser
```

Use context:
```
$ k config use-context myuser-context
```

Try to get list of Pods (an error is expected):
```
$ k get pod
Error from server (Forbidden): pods is forbidden: User "myuser" cannot list resource "pods" in API group "" in the namespace "default"
```

Switch back to user that you used before, e.g.:
```
$ k config use-context kubernetes-admin@kubernetes
```


Create clustert role:
```
$ kubectl create clusterrole pod-reader --verb=get,list,watch --resource=pod
```

Bind cluster role:
```
$ kubectl create clusterrolebinding myrolebinding --clusterrole=pod-reader --group=myusergroup
```

Try to get list of Pods (no error is expected):
```
$ k get pod
NAME            READY   STATUS      RESTARTS   AGE
<output trunkated>
```

