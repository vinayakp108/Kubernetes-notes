<h1>TLS Basics<br><br></h1>

<h2> 1️⃣ TLS म्हणजे काय?<br></h2>
TLS म्हणजे Transport Layer Security. याचा उद्देश data encryption, authentication आणि integrity सुनिश्चित करणं आहे.<br>
Kubernetes मध्ये TLS वापरून:- <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&bull;  API server आणि clients (kubectl, kubelets) यांच्यातील communication secure होते.<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&bull;  Cluster मध्ये data transmission (etcd, kube-apiserver, kubelet, ingress) encrypted राहतो.<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&bull;  Unauthorized access (man-in-the-middle attack) टाळता येतो.<br>
<br>
<h2> 2️⃣ TLS certificate types<br></h2>
1) CA certificate (Root) :- <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&bull;  Cluster trusted authority. सर्व other certificates verify करण्यासाठी.<br>
2) Server certificate :- <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&bull; Server identify करण्यासाठी (API server, Ingress, etcd). <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&bull; Subject = server name / IP.<br>
3) Client certificate :-  <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&bull; Client identity verify करण्यासाठी (kubelet, kubectl).<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&bull;Subject = username / system name.<br>
4) Etcd peer certificates (mutual TLS) :- <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&bull; Etcd nodes एकमेकांशी communicate करताना authenticate करतात.<br>
<br><br>
<h2> 3️⃣ Kubernetes मधील TLS workflow<br></h2>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&bull; Main workflow: Certificate Authority (CA) तयार करणे<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&bull; Cluster मध्ये आपली स्वतःची CA असते जी certificates issue करते.<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&bull; CA public/private key pair generate करते.<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&bull;  प्रत्येक component (API server, kubelet, etcd, ingress) साठी certificate तयार करतो.<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&bull; Certificate मध्ये subject (component name), validity, CA signature असते.<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&bull; Certificate distribute करणे: Each component ला certificate आणि private key configure करतो.<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&bull; उदा: kube-apiserver ला server certificate आणि CA certificate configure करतो.<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&bull; TLS handshake सुरू होते: Client (kubectl, kubelet) API server शी connect करताना certificate verify करतो.<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&bull; Server client certificate verify करू शकतो (mutual TLS).<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&bull; Encrypted communication: Data encrypted म्हणून transit मध्ये secure राहते.<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&bull; Message integrity आणि authentication verify होतो.<br>
<br>
    <h2>4️⃣ CA Certificate का ठेवावं लागतं?</h2>
    <ul>
      <li>CA (Certificate Authority) म्हणजे एक trusted entity जी दुसऱ्या certificates sign करते.</li>
      <li>Client कडे CA चा public certificate असतो.</li>
      <li>Server चा certificate verify करण्यासाठी client CA चा cert वापरतो.</li>
      <li>Trust chain establish करण्यासाठी CA certificate लागतो.</li>
      <li>Without CA: client ला कळणार नाही की server चा cert खरोखर valid आहे का.</li>
    </ul>
  </div>
<br>
  <div class="section">
    <h2>5️⃣ CSR + Key file सोबत CA certificate का mention करतो?</h2>
    <p><strong>CSR (Certificate Signing Request) कसं काम करतं:</strong></p>
    <ul>
      <li>CSR मध्ये तुमचा public key + identity (CN, O, etc.) पाठवतो.</li>
      <li>CA हे CSR verify करून certificate issue करतं.</li>
    </ul>
    <p><strong>Key file कसा वापरला जातो:</strong></p>
    <ul>
      <li>Private key तुमचं असतं — जे CSR तयार करताना वापरतो.</li>
      <li>TLS handshake दरम्यान identity prove करण्यासाठी वापरतो.</li>
    </ul>
    <p><strong>CA cert सोबत CSR & key का mention करतो?</strong></p>
    <ul>
      <li>CSR: CA ला सांगतो "मला हे cert हवंय, ही माझी identity."</li>
      <li>Key: तुमचा private key — जो certificate वापरण्यासाठी लागतो.</li>
      <li>CA cert: Trust establish करण्यासाठी — जो client verify करू शकतो.</li>
    </ul>
  </div>
<br>
  <div class="section">
    <h2>6️⃣ TLS Certificate Workflow Summary</h2>
    <ul>
      <li>CSR तयार करणं</li>
      <li>Private key generate करणं</li>
      <li>CA ने certificate sign करणं</li>
      <li>Certificate configure करणं</li>
      <li>TLS handshake आणि encrypted communication</li>
    </ul>
  </div>
<br>
  <div class="section">
    <h2>7️⃣ Real-world Analogy</h2>
    <ul>
      <li>CSR: Job application — तुम्ही सांगता "ही माझी qualification आहे."</li>
      <li>CA: Employer — verify करून तुम्हाला job (certificate) देतो.</li>
      <li>Key: तुमचा signature — जो तुम्ही वापरता identity prove करण्यासाठी.</li>
      <li>CA cert: Reference letter — जो दुसरे लोक trust करू शकतात.</li>
    </ul>
  </div>

</body>
</html>


