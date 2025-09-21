
# 📘 DevOps Learning Notes: APIs and Kubernetes API

This file contains structured notes from a learning conversation about APIs, how they work, and how Kubernetes exposes its API.  

---

## 1. What is an API?

- **API = Application Programming Interface**
- Acts like a **waiter** between client and server:
  - Client (you) makes a request
  - API (waiter) passes it to the system (kitchen)
  - System prepares data and API returns the response

👉 Used everywhere in DevOps:
- Kubernetes (kubectl → API)
- Jenkins, GitHub, ArgoCD
- Cloud providers (AWS, Azure, GCP)

---

## 2. How APIs Work

1. **Client → Request**
   - Method (`GET`, `POST`, `PUT`, `DELETE`)
   - Headers (auth, content type)
   - Body (data, usually JSON)

2. **API Server → Process**
   - Checks authentication, authorization, validation
   - Executes business logic

3. **Server → Response**
   - Status code (200, 404, 500)
   - Headers
   - Body (JSON/XML)

Example GitHub API call:
```bash
curl https://api.github.com/repos/kubernetes/kubernetes
```

Response (trimmed):
```json
{
  "name": "kubernetes",
  "language": "Go",
  "stargazers_count": 111000
}
```

---

## 3. Build a Simple API (Python Flask Example)

Install Flask:
```bash
pip install flask
```

**Code (`app.py`):**
```python
from flask import Flask, request, jsonify
app = Flask(__name__)

@app.route('/hello', methods=['GET'])
def hello():
    return jsonify({"message": "Hello, DevOps Learner!"})

@app.route('/user/<username>', methods=['GET'])
def get_user(username):
    return jsonify({"user": username, "role": "DevOps Engineer"})

@app.route('/deploy', methods=['POST'])
def deploy():
    data = request.json
    return jsonify({
        "status": "success",
        "service": data.get("service"),
        "version": data.get("version")
    })

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

Run:
```bash
python app.py
```

Test with curl:
```bash
curl http://127.0.0.1:5000/hello
curl http://127.0.0.1:5000/user/rohith
curl -X POST http://127.0.0.1:5000/deploy -H "Content-Type: application/json" -d '{"service":"nginx","version":"1.21"}'
```

---

## 4. Kubernetes API

Kubernetes is built around a **REST API server (`kube-apiserver`)**.

**API format:**
```
https://<kube-apiserver>/apis/<group>/<version>/<resource>
```

Examples:
- Pods: `/api/v1/namespaces/default/pods`
- Deployments: `/apis/apps/v1/namespaces/default/deployments`
- Nodes: `/api/v1/nodes`

### Example: Get Pods
When you run:
```bash
kubectl get pods -n default
```
It actually calls:
```http
GET /api/v1/namespaces/default/pods
Authorization: Bearer <token>
Accept: application/json
```

Response (trimmed):
```json
{
  "kind": "PodList",
  "items": [
    {
      "metadata": {"name": "nginx-deployment-66b6c48dd5-abcde"},
      "status": {"phase": "Running"}
    }
  ]
}
```

---

## 5. Accessing Kubernetes API

### Method 1: Proxy
```bash
kubectl proxy
curl http://127.0.0.1:8001/api/v1/namespaces/default/pods
```

### Method 2: Direct API with Token
```bash
APISERVER=$(kubectl config view -o jsonpath='{.clusters[0].cluster.server}')
TOKEN=$(kubectl get secret $(kubectl get sa default -o jsonpath='{.secrets[0].name}') -o jsonpath='{.data.token}' | base64 --decode)

curl -k $APISERVER/api/v1/namespaces/default/pods   --header "Authorization: Bearer $TOKEN"
```

---

## 6. Create a Pod via API

**Manifest (pod.json):**
```json
{
  "apiVersion": "v1",
  "kind": "Pod",
  "metadata": {
    "name": "nginx-api",
    "namespace": "default"
  },
  "spec": {
    "containers": [
      {
        "name": "nginx",
        "image": "nginx:latest",
        "ports": [
          {"containerPort": 80}
        ]
      }
    ]
  }
}
```

**POST request:**
```bash
kubectl proxy

curl -X POST http://127.0.0.1:8001/api/v1/namespaces/default/pods   -H "Content-Type: application/json"   -d @pod.json
```

Response (trimmed):
```json
{
  "kind": "Pod",
  "metadata": {"name": "nginx-api", "namespace": "default"},
  "status": {"phase": "Pending"}
}
```

---

# ✅ Key Takeaways

- APIs = backbone of DevOps automation.  
- Kubernetes API is just HTTP + JSON.  
- `kubectl`, Terraform, Jenkins → all clients making API calls.  
- You can directly **GET, POST, PATCH, DELETE** resources via the API.  


