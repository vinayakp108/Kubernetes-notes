<h1>Kubernetes TLS Certificates Workflow (Production-ready)<br><br></h1>


<h2>1. Root Certificate Authority (CA)<br></h2>
Explanation:-<br>
&bull; Master node वर root CA तयार करा.<br>
&bull; Root CA server certificate + private key सुरक्षित ठेवा.<br>
&bull; सर्व cluster certificates साठी signing authority म्हणून वापरली जाईल.<br><br>

```
# Master node वर root CA तयार करा
openssl genrsa -out ca.key 4096
openssl req -x509 -new -nodes -key ca.key -subj "/CN=kubernetes-ca" -days 3650 -out ca.crt
```
<br>

<h2>2. API Server Certificates<br></h2>
Explanation:-<br>
&bull; API server साठी server certificate आणि key तयार करा.<br>
&bull; Alternate DNS/IP names include करणे mandatory आहे.<br>
&bull; Signed by Root CA.<br><br>

```
# OpenSSL configuration file तयार करा (openssl.cnf)
cat <<EOF > openssl.cnf
[ req ]
default_bits = 2048
prompt = no
distinguished_name = dn
req_extensions = req_ext

[ dn ]
CN = kube-apiserver

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = kubernetes
DNS.2 = kubernetes.default
DNS.3 = kubernetes.default.svc
DNS.4 = kubernetes.default.svc.cluster.local
IP.1 = 10.96.0.1
EOF

# API server key तयार करा
openssl genrsa -out apiserver.key 2048

# CSR तयार करा
openssl req -new -key apiserver.key -out apiserver.csr -config openssl.cnf

# Certificate signed by CA
openssl x509 -req -in apiserver.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out apiserver.crt -days 365 -extensions req_ext -extfile openssl.cnf
```
<br>


<h2>3. ETCD Certificates<br></h2>
Mutual TLS (mTLS) काय देते?<br>
&bull; Server प्रमाणित करतो client<br>
&bull; Client प्रमाणित करतो server<br>
&bull; दोन्ही बाजू एकमेकांच्या identity verify करतात. <br>
&bull; Network मधून येणारी सगळी data encrypt होते<br>
&bull; Phishing / Man-in-the-middle attack रोखतो<br>
म्हणजे एकच certificate वापरून client आणि server दोघांचं verify करणं शक्य नाही, त्यासाठी दोन्ही बाजूंना वेगवेगळे certificates लागतात.<br>
Production cluster मध्ये etcd, kube-apiserver, kubelet, admin clients साठी mTLS setup करणे best practice आहे.<br>
<br>
Explanation:-<br>
&bull; etcd server आणि client certificates generate करा.<br>
&bull; API server etcd शी communicate करताना client certificate वापरेल.<br><br>

```
# etcd server key & CSR
openssl genrsa -out etcd-server.key 2048
openssl req -new -key etcd-server.key -subj "/CN=etcd-server" -out etcd-server.csr

# etcd server certificate signed by CA
openssl x509 -req -in etcd-server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out etcd-server.crt -days 365

# etcd client key & CSR
openssl genrsa -out etcd-client.key 2048
openssl req -new -key etcd-client.key -subj "/CN=apiserver-etcd-client" -out etcd-client.csr

# etcd client certificate signed by CA
openssl x509 -req -in etcd-client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out etcd-client.crt -days 365
```
<br>

<h2>4. Kubelet Certificates<br></h2>
Explanation:-<br>
&bull; Each worker node kubelet साठी key + certificate generate करा.<br>
&bull; API server kubelet verify करण्यासाठी client certificate वापरेल.<br><br>

```
# Kubelet key & CSR
openssl genrsa -out kubelet.key 2048
openssl req -new -key kubelet.key -subj "/CN=system:node:worker1" -out kubelet.csr

# Kubelet certificate signed by CA
openssl x509 -req -in kubelet.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out kubelet.crt -days 365
```
<br>

<h2>5. Admin / Scheduler / Kube-proxy Certificates<br></h2>
Explanation:-<br>
&bull; Admin, Scheduler, Kube-proxy client certificates generate करा.<br>
&bull; API server authenticate करण्यासाठी certificates वापरतील.<br><br>

```
# Admin key & CSR
openssl genrsa -out admin.key 2048
openssl req -new -key admin.key -subj "/CN=admin/O=system:masters" -out admin.csr
openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out admin.crt -days 365

# Scheduler key & certificate
openssl genrsa -out scheduler.key 2048
openssl req -new -key scheduler.key -subj "/CN=system:kube-scheduler" -out scheduler.csr
openssl x509 -req -in scheduler.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out scheduler.crt -days 365

# Kube-proxy key & certificate
openssl genrsa -out kube-proxy.key 2048
openssl req -new -key kube-proxy.key -subj "/CN=system:kube-proxy" -out kube-proxy.csr
openssl x509 -req -in kube-proxy.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out kube-proxy.crt -days 365
```
<br>
<h2>6. Certificate Placement & Configuration<br></h2>
Explanation:-<br>
&bull; Master node: API server + CA + etcd client certificates<br>
&bull; Worker nodes: Kubelet certificates<br>
&bull; Admin client: admin.crt + admin.key<br>
&bull; Scheduler/Kube-proxy: client certificates<br>
Mandatory: CA certificate (ca.crt) सर्व nodes/clients वर copy करा.<br><br>

```
# Example master node paths
sudo mkdir -p /etc/kubernetes/pki
sudo cp ca.crt apiserver.crt apiserver.key /etc/kubernetes/pki/
sudo cp etcd-server.crt etcd-server.key etcd-client.crt etcd-client.key /etc/kubernetes/pki/etcd/
```
```
# Example worker node paths
sudo mkdir -p /var/lib/kubelet/pki
sudo cp ca.crt kubelet.crt kubelet.key /var/lib/kubelet/pki/
```
```
# Admin client paths
mkdir -p ~/.kube
cp ca.crt admin.crt admin.key ~/.kube/
```
<br>

<h2>7. Verification<br></h2>
Explanation:-<br>
&bull; सर्व server आणि client certificates verify करा.<br>
&bull; Unauthorized / expired certificates deny करा.<br><br>

```
# Verify certificate
openssl verify -CAfile ca.crt apiserver.crt
openssl verify -CAfile ca.crt etcd-server.crt
openssl verify -CAfile ca.crt kubelet.crt
```
<br>

</h2>💡 Scenario Example (Production-ready)<br></h2>

Master node: API server serving certificate + CA + etcd client<br>

Worker nodes: Kubelet certificates<br>

Users: Admin certificate for kubectl access<br>

Components: Scheduler/Kube-proxy client certificates<br>

Verification: openssl verify command with CA certificate<br>


