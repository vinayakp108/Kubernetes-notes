<h3>kubectl proxy म्हणजे काय आणि आपण का वापरतो Kubernetes मध्ये 👇</h3><br>

<h4>🔹 kubectl proxy म्हणजे काय?</h4>

<p>
kubectl proxy हा एक <strong>स्थानिक (local) HTTP proxy server</strong> आहे  जो <strong>Kubernetes API server</strong> शी सुरक्षितपणे (authenticated) संपर्क साधतो.
kubectl proxy म्हणजे Kubernetes API server कडे जाणारा एक सुरक्षित स्थानिक मार्ग (local gateway) आहे,जो authentication आणि encryption आपोआप handle करतो.
याचा वापर dashboard, testing आणि internal API calls साठी सुरक्षितपणे केला जातो.<br><br>

हा proxy तुझ्या <code>kubeconfig</code> फाइलमधील credentials वापरतो  आणि authentication व TLS internally handle करतो.<br><br>
</p>

<h4>🔹 का वापरतो?</h4>

<p>
<strong>API Server सुरक्षितपणे access करण्यासाठी</strong><br>
→ तुझ्या cluster चा API server direct public network वर उघडायचा नसतो.<br>
→ म्हणून <strong>kubectl proxy</strong> वापरून तू local gateway तयार करतोस.<br><br>

<strong>Authentication आपोआप होते</strong><br>
→ proxy तुझ्या kubeconfig मधून certificate/token वापरून API server शी authenticated request पाठवतो.<br>
→ तुला manually काही token/cert देण्याची गरज नसते.<br><br>

<strong>Dashboard सुरक्षितपणे access करण्यासाठी</strong><br>
→ kubectl proxy वापरून तू Kubernetes Dashboard उघडू शकतोस 👇<br><br>
</p>

<p>
— आणि त्यासाठी extra TLS/key configure करायची गरज नसते.<br><br>

<strong>API endpoints test/debug करण्यासाठी</strong><br>
→ Proxy वापरून तू <code>curl</code> / <code>browser</code> मधून cluster च्या REST endpoints check करू शकतोस.<br><br>

<strong>Cluster बाहेरून expose न करता internal access मिळवण्यासाठी</strong><br>
→ हे सुरक्षित आहे, कारण proxy तुझ्या local machine वरच चालतो (default port <strong>8001</strong>)  आणि बाहेरच्या world ला cluster उघडत नाही.<br>
</p> 

```
# Proxy सुरू करा
kubectl proxy
``` 
<h4>🔹Starting to serve on 127.0.0.1:8001 #output</h4> <br>

```
curl http://127.0.0.1:8001/api/v1/namespaces/default/pods
```
<h4>🔹 Proxy तुझ्या credentials वापरून API server शी authenticated request पाठवतो आणि तुला pods ची यादी परत मिळते.</h4>












