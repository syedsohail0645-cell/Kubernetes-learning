1️⃣ What is an API request in Kubernetes?

An API request is simply:

A request sent to the kube-apiserver to read, create, update, or delete a Kubernetes resource.

Everything you do with kubectl becomes an HTTP request.

2️⃣ High-level flow (big picture)
kubectl  ──► kube-apiserver ──► etcd
              │
              ├─ AuthN
              ├─ AuthZ (RBAC)
              ├─ Admission Controllers
              └─ API handling


You talk to kubectl, but kubectl talks to the API server.

3️⃣ Full Anatomy of an API Request (Core concept)

A Kubernetes API request has 5 main parts:

<HTTP VERB> <API PATH> <RESOURCE> <NAMESPACE> <BODY>


Let’s break each one.

1️⃣ HTTP Verb (WHAT action?)
Verb	Meaning
GET	Read
POST	Create
PUT	Replace
PATCH	Update
DELETE	Remove

Example:

GET

2️⃣ API Path (WHERE?)

This is where API groups and versions come in.

Core API group:
/api/v1

Named API group:
/apis/<group>/<version>


Example:

/apis/apps/v1

3️⃣ Resource (WHAT object?)

Plural, lowercase:

pods
deployments
services


Example:

/apis/apps/v1/deployments

4️⃣ Namespace (OPTIONAL)

Namespaced resources need it

Cluster-wide resources don’t

Namespaced:
/namespaces/default/pods

Cluster-wide:
/nodes

5️⃣ Request Body (DATA)

Only needed for:

POST

PUT

PATCH

This is your YAML converted to JSON.

4️⃣ Complete API Request Examples
Example 1: Get all Pods
kubectl get pods


Actual API call:

GET /api/v1/namespaces/default/pods


Breakdown:

GET              → Read
/api/v1          → Core API group + version
namespaces/default → Namespace
pods             → Resource

Example 2: Create a Deployment
kubectl apply -f deploy.yaml


API request:

POST /apis/apps/v1/namespaces/default/deployments


Body:

{
  "apiVersion": "apps/v1",
  "kind": "Deployment",
  ...
}

Example 3: Delete a Service
kubectl delete svc mysvc


API request:

DELETE /api/v1/namespaces/default/services/mysvc

5️⃣ How YAML maps to API request 🧠💡

YAML:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: default


Maps to:

/apis/apps/v1/namespaces/default/deployments/myapp

6️⃣ What kube-apiserver does with the request

Once the request arrives:

Authentication – Who are you?

Authorization (RBAC) – Are you allowed?

Admission Controllers – Is the request valid?

Persist to etcd – Save state

Notify controllers – Reconcile desired state

7️⃣ One simple mental model (remember forever)
kubectl → API request → kube-apiserver → etcd


And the request is built using:

Verb + Group + Version + Resource + Namespace

8️⃣ Ultra-short interview explanation

A Kubernetes API request is an HTTP request sent to the kube-apiserver using a specific API group, version, resource, and namespace, which is then authenticated, authorized, validated, and stored in etcd.


================================================================================================================

Example's
==========

🔹 SECTION 1: Creating & Reading a Pod
1️⃣ Create a Pod using YAML
kubectl apply -f pod.yaml


What you are doing

You are asking Kubernetes to create or update a Pod defined in pod.yaml.

Behind the scenes (API)

kubectl reads YAML

Converts YAML → JSON

Sends API request:

POST /api/v1/namespaces/default/pods


Why POST?

Because you are creating a resource.

Key takeaway

kubectl apply → API server → stores Pod in etcd

2️⃣ Get a specific Pod
kubectl get pod hello-world


What you are doing

Asking Kubernetes to fetch details of one Pod.

API request

GET /api/v1/namespaces/default/pods/hello-world


Key takeaway

GET = read-only request

🔹 SECTION 2: Verbosity (-v) Levels (VERY IMPORTANT)

Verbosity shows how kubectl talks to the API server.

3️⃣ Verbosity level 6
kubectl get pod hello-world -v 6


What you see

HTTP method (GET)

API URL

Response code (200, 404, etc.)

Example output snippet:

GET https://<api-server>/api/v1/namespaces/default/pods/hello-world
Response Status: 200 OK


Focus on

VERB → GET

API Path → /api/v1/...

Response code → 200 OK

4️⃣ Verbosity level 7
kubectl get pod hello-world -v 7


Adds

HTTP request headers

Example:

User-Agent: kubectl/v1.29
Accept: application/json


Why this matters

Shows kubectl is just an HTTP client

Confirms API uses JSON

5️⃣ Verbosity level 8
kubectl get pod hello-world -v 8


Adds

Response headers

Truncated response body

Example:

Content-Type: application/json


Meaning

API server responds with JSON metadata

6️⃣ Verbosity level 9
kubectl get pod hello-world -v 9


Adds

FULL response body

You’ll see:

metadata:
  name: hello-world
  namespace: default
  resourceVersion: "12345"


Key thing to notice

metadata → stored in etcd

resourceVersion → used for watches

🔹 SECTION 3: kubectl proxy (Direct API Access)
7️⃣ Start proxy
kubectl proxy &


What happens

Starts local HTTP proxy at:

http://localhost:8001


Why

Automatically authenticates you

Lets you access API server using curl

8️⃣ Direct API call
curl http://localhost:8001/api/v1/namespaces/default/pods/hello-world | head -n 10


What this proves

Kubernetes API is REST

kubectl is optional

You can use curl

API endpoint

GET /api/v1/namespaces/default/pods/hello-world

9️⃣ Bring proxy to foreground & stop
fg
ctrl+c


Stops kubectl proxy.

🔹 SECTION 4: Watch Requests (Streaming API)
🔟 Watch Pods
kubectl get pods --watch -v 6 &


What this does

Opens persistent connection

Watches for changes

API request

GET /api/v1/namespaces/default/pods?watch=true


Important

Uses resourceVersion

Keeps TCP connection open

1️⃣1️⃣ Check TCP connection
netstat -plant | grep kubectl


What you see

kubectl has an open TCP connection

Confirms watch is streaming data

1️⃣2️⃣ Delete Pod (while watching)
kubectl delete pods hello-world


API request

DELETE /api/v1/namespaces/default/pods/hello-world


What happens

Watch immediately prints event

Watch remains active

1️⃣3️⃣ Re-create Pod
kubectl apply -f pod.yaml


Watch prints ADDED event.

1️⃣4️⃣ Stop watch
fg
ctrl+c

🔹 SECTION 5: Logs API
1️⃣5️⃣ Fetch logs
kubectl logs hello-world


API request

GET /api/v1/namespaces/default/pods/hello-world/log

1️⃣6️⃣ Logs with verbosity
kubectl logs hello-world -v 6


You’ll see the exact log endpoint URL.

1️⃣7️⃣ Logs via proxy
kubectl proxy &
curl http://localhost:8001/api/v1/namespaces/default/pods/hello-world/log


Key lesson

Logs are also just API requests

🔹 SECTION 6: Authentication Failure Demo
1️⃣8️⃣ Backup kubeconfig
cp ~/.kube/config ~/.kube/config.ORIG


Safety backup.

1️⃣9️⃣ Break authentication
vi ~/.kube/config


Change username → invalid user.

2️⃣0️⃣ Try API access
kubectl get pods -v 6


API response

403 Forbidden


Meaning

Authentication failed

API server rejected request

2️⃣1️⃣ Restore kubeconfig
cp ~/.kube/config.ORIG ~/.kube/config


Access restored.

🔹 SECTION 7: Missing Resources (404)
2️⃣2️⃣ Non-existent Pod
kubectl get pods nginx-pod -v 6


API response

404 Not Found


Meaning

API path valid

Resource name does not exist

🔹 SECTION 8: Deployment Lifecycle
2️⃣3️⃣ Create Deployment
kubectl apply -f deployment.yaml -v 6


kubectl first checks:

GET /apis/apps/v1/deployments/hello-world → 404


Then creates:

POST /apis/apps/v1/namespaces/default/deployments → 201 Created

2️⃣4️⃣ Get deployments
kubectl get deployment

GET /apis/apps/v1/deployments

2️⃣5️⃣ Delete Deployment
kubectl delete deployment hello-world -v 6

DELETE /apis/apps/v1/namespaces/default/deployments/hello-world

2️⃣6️⃣ Delete Pod
kubectl delete pod hello-world

DELETE /api/v1/namespaces/default/pods/hello-world

🧠 FINAL BIG PICTURE (MEMORIZE THIS)

Every kubectl command becomes:

HTTP VERB + API GROUP + VERSION + RESOURCE + NAMESPACE


Examples:

GET    /api/v1/pods
POST   /apis/apps/v1/deployments
DELETE /api/v1/pods/hello-world

