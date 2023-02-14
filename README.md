# Istio

Istio helm for NIC use-cases

Prerequisites:
1. A kubernetes on cloud cluster
2. Istioctl
3. HELM


Use cases:
1. Traffic Management: Solution should separate traffic management from infrastructure scaling (which is handled by Kubernetes). This separation allows for features that can live outside the application code,
1. Like dynamic request routing for A/B testing
2. gradual rollouts
3. canary releases
4. retries
5. circuit breakers
6. fault injection.



2. Fault injection: Solution should provide protocol-specific fault injection into the network.
3. Load balancing: Solution should support L7 (application layer) based load balancing modes (round robin, random and weighted least request.
4. Backend abstraction: Solution should provide component that provides policy control and telemetry collection.
5. Service authentication: Solution should make sure that all the services can only be accessed from strongly authenticated and authorised clients.
6. Role-based access control: Solution should support namespace-level, service-level and method-level access control for services.
7. Support for multi cluster communication


Task Performed:

1. export  KUBECONFIG=< your kubeconfig >
2. Create a namespace

`kubectl create ns istio-system`


Clone istio repo

```
git clone https://github.com/istio/istio.git

cd istio/manifests/charts
```

4. Deploy istio base, istiod and gateway 

```
helm install base base -n istio-system 

helm install istiod istiod -n istio-system 

helm install ingressgateway gateway  -n istio-system 
```

5. Check the pods of istio-system
```
kubectl get po -n istio-system
kubectl get svc -n istio-system
```

6. Label the namespace, to make the istio sidecar pods injection with the application pods for envoy proxy, enabling istio with your application

```
kubectl label namespace <namespace> istio-injection=enabled
```

7.Deploy bookinfo application

```
apiVersion: v1
kind: Service
metadata:
  name: details
  labels:
    app: details
    service: details
spec:
  ports:
  - port: 9080
    name: http
  selector:
    app: details
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: bookinfo-details
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: details-v1
  labels:
    app: details
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: details
      version: v1
  template:
    metadata:
      labels:
        app: details
        version: v1
    spec:
      serviceAccountName: bookinfo-details
      containers:
      - name: details
        image: docker.io/istio/examples-bookinfo-details-v1:1.15.0
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 9080
---
##################################################################################################
# Ratings service
##################################################################################################
apiVersion: v1
kind: Service
metadata:
  name: ratings
  labels:
    app: ratings
    service: ratings
spec:
  ports:
  - port: 9080
    name: http
  selector:
    app: ratings
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: bookinfo-ratings
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ratings-v1
  labels:
    app: ratings
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ratings
      version: v1
  template:
    metadata:
      labels:
        app: ratings
        version: v1
    spec:
      serviceAccountName: bookinfo-ratings
      containers:
      - name: ratings
        image: docker.io/istio/examples-bookinfo-ratings-v1:1.15.0
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 9080
---
##################################################################################################
# Reviews service
##################################################################################################
apiVersion: v1
kind: Service
metadata:
  name: reviews
  labels:
    app: reviews
    service: reviews
spec:
  ports:
  - port: 9080
    name: http
  selector:
    app: reviews
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: bookinfo-reviews
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reviews-v1
  labels:
    app: reviews
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: reviews
      version: v1
  template:
    metadata:
      labels:
        app: reviews
        version: v1
    spec:
      serviceAccountName: bookinfo-reviews
      containers:
      - name: reviews
        image: docker.io/istio/examples-bookinfo-reviews-v1:1.15.0
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 9080
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reviews-v2
  labels:
    app: reviews
    version: v2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: reviews
      version: v2
  template:
    metadata:
      labels:
        app: reviews
        version: v2
    spec:
      serviceAccountName: bookinfo-reviews
      containers:
      - name: reviews
        image: docker.io/istio/examples-bookinfo-reviews-v2:1.15.0
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 9080
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reviews-v3
  labels:
    app: reviews
    version: v3
spec:
  replicas: 1
  selector:
    matchLabels:
      app: reviews
      version: v3
  template:
    metadata:
      labels:
        app: reviews
        version: v3
    spec:
      serviceAccountName: bookinfo-reviews
      containers:
      - name: reviews
        image: docker.io/istio/examples-bookinfo-reviews-v3:1.15.0
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 9080
---
##################################################################################################
# Productpage services
##################################################################################################
apiVersion: v1
kind: Service
metadata:
  name: productpage
  labels:
    app: productpage
    service: productpage
spec:
  ports:
  - port: 9080
    name: http
  selector:
    app: productpage
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: bookinfo-productpage
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: productpage-v1
  labels:
    app: productpage
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: productpage
      version: v1
  template:
    metadata:
      labels:
        app: productpage
        version: v1
    spec:
      serviceAccountName: bookinfo-productpage
      containers:
      - name: productpage
        image: docker.io/istio/examples-bookinfo-productpage-v1:1.15.0
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 9080
```


8. Apply the above bookinfodeploy.yaml
```
kubectl apply -f bookinfodeploy.yaml
```
9. Check the pods

```
kubectl get po
```

10. Confirm that the Bookinfo application is running, send a request to it by a curl command from some pod, for example from ratings:
```
kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"
```

11. Create Istio gateway
```
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: bookinfo-gateway
spec:
  selector:
    istio: ingressgateway # use istio default controller
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
  - "*"
  gateways:
  - bookinfo-gateway
  http:
  - match:
    - uri:
        exact: /productpage
    - uri:
        prefix: /static
    - uri:
        exact: /login
    - uri:
        exact: /logout
    - uri:
        prefix: /api/v1/products
    route:
    - destination:
        host: productpage
        port:
          number: 9080
```


12. Apply the gateway file

```
kubectl apply -f bookinfogateway.yaml
```

13. Export the values of INGRESS_HOST, INGRESS_PORT

```
export INGRESS_NAME=<Ingress name under istio-system.service.ingressname>
export INGRESS_NS=istio-system

export INGRESS_HOST=$(kubectl -n "$INGRESS_NS" get service "$INGRESS_NAME" -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

export INGRESS_PORT=$(kubectl -n "$INGRESS_NS" get service "$INGRESS_NAME" -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
```

14. Set gateway URL
```
export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
```

15. Application is accessible from outside the cluster
```
curl -s "http://${GATEWAY_URL}/productpage" | grep -o "<title>.*</title>"
```

16. Putting weight on the application

```
while sleep 0.01;do curl -sS 'http://'"$INGRESS_HOST"':'"$INGRESS_PORT"'/productpage'\ &> /dev/null ; done
```

17. Istio uses subsets, in destination rules, to define versions of a service. Run the following command to create default destination rules for the Bookinfo services:

```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: productpage
spec:
  host: productpage
  subsets:
  - name: v1
    labels:
      version: v1
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
  - name: v3
    labels:
      version: v3
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: ratings
spec:
  host: ratings
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
  - name: v2-mysql
    labels:
      version: v2-mysql
  - name: v2-mysql-vm
    labels:
      version: v2-mysql-vm
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: details
spec:
  host: details
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2

```

a. Apply the above destinationrule.yaml
```
kubectl apply -f destination-rule.yaml
```

18. Request routing

a. The initial goal of this task is to apply rules that route all traffic to v1 (version 1) of the microservices. Later, you will apply a rule to route traffic based on the value of an HTTP request header. no review appear


```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: productpage
spec:
  hosts:
  - productpage
  http:
  - route:
    - destination:
        host: productpage
        subset: v1
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - route:
    - destination:
        host: ratings
        subset: v1
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: details
spec:
  hosts:
  - details
  http:
  - route:
    - destination:
        host: details
        subset: v1
---

```

b. Apply the req routing yaml

```
kubectl apply -f request-routing.yaml
```

19. Weight based routing

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: productpage
spec:
  hosts:
  - productpage
  http:
  - route:
    - destination:
        host: productpage
        subset: v1
      weight: 50
```
a. Apply the above file
```
kubectl apply -f reqrouting2.yaml
```


20. Test the new routing configuration

a. Open the Bookinfo site in your browser. The URL is http://$GATEWAY_URL/productpage, where $GATEWAY_URL is the External IP address of the ingress,

b. Reviews part of the page displays with no rating stars, no matter how many times you refresh.


21. Route based on user identity
a. Change the route configuration so that all traffic from a specific user is routed to a specific service version, reviews will appear

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
    - reviews
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    route:
    - destination:
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v1
```
```
kubectl apply -f changereqroute.yaml
```

23. Fault Injection

a. How to inject faults to test the resiliency of your application.
b. Inject Fault flag in virtual service yaml for a specific subset/user.
c. Injecting an HTTP delay fault


```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    fault:
      delay:
        percentage:
          value: 100.0
        fixedDelay: 7s
    route:
    - destination:
        host: ratings
        subset: v1
  - route:
    - destination:
        host: ratings
        subset: v1

```

d. Apply the above file
```
kubectl apply -f fault-injection.yaml
```
f. Testing the delay configuration

g. Expect the Bookinfo home page to load without errors in approximately 7 seconds for the user jason. However, there is a problem: the Reviews section displays an error message


24. Traffic Shifting

a. How to gradually migrate traffic from one version of a microservice to another

b. Migrate traffic gradually from one version of a microservice to another

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: productpage
spec:
  hosts:
  - productpage
  http:
  - route:
    - destination:
        host: productpage
        subset: v1
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - route:
    - destination:
        host: ratings
        subset: v1
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: details
spec:
  hosts:
  - details
  http:
  - route:
    - destination:
        host: details
        subset: v1
---


```


c.  Apply traffic-shifting.yaml
```
kubectl apply -f traffic-shifting.yaml
```
d. Create another yaml for shifting traffic

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
    - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
      weight: 50
    - destination:
        host: reviews
        subset: v3
      weight: 50

```


e.  Apply traffic-shifting1.yaml
```
kubectl apply -f traffic-shifting1.yaml
```

f. Migrated traffic from an old to new version of the reviews service using Istio’s weighted routing feature.


25. Circuit breaking

a. Circuit breaking allows you to write applications that limit the impact of failures, latency spikes, and other undesirable effects of network peculiarities.

b. Create a sample application deploy 'httpbin.yaml'

```
# httpbin service
##################################################################################################
apiVersion: v1
kind: ServiceAccount
metadata:
  name: httpbin
---
apiVersion: v1
kind: Service
metadata:
  name: httpbin
  labels:
    app: httpbin
    service: httpbin
spec:
  ports:
  - name: http
    port: 8000
    targetPort: 80
  selector:
    app: httpbin
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpbin
spec:
  replicas: 1
  selector:
    matchLabels:
      app: httpbin
      version: v1
  template:
    metadata:
      labels:
        app: httpbin
        version: v1
    spec:
      serviceAccountName: httpbin
      containers:
      - image: docker.io/kennethreitz/httpbin
        imagePullPolicy: IfNotPresent
        name: httpbin
        ports:
        - containerPort: 80

```

```
kubectl apply -f httpbin.yaml
```

d. Create circuit breaker configure them in destination rule

```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: httpbin
spec:
  host: httpbin
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 1
      http:
        http1MaxPendingRequests: 1
        maxRequestsPerConnection: 1
    outlierDetection:
      consecutive5xxErrors: 1
      interval: 1s
      baseEjectionTime: 3m
      maxEjectionPercent: 100



```

```
kubectl apply -f httpbin_destinationrule.yaml
```

f. Verify the destination rule was created correctly
 ```
 kubectl get destinationrule httpbin -o yaml
 ```
 
 g. Adding a client
 
i. Create a client to send traffic to the httpbin service. The client is a simple load-testing client called fortio. Fortio lets you control the number of connections, concurrency, and delays for outgoing HTTP calls. You will use this client to “trip” the circuit breaker policies you set in the DestinationRule. 

ii. Inject the client with the Istio sidecar proxy so network interactions are governed by Istio.

```
apiVersion: v1
kind: Service
metadata:
  name: fortio
  labels:
    app: fortio
    service: fortio
spec:
  ports:
  - port: 8080
    name: http
  selector:
    app: fortio
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fortio-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: fortio
  template:
    metadata:
      annotations:
        # This annotation causes Envoy to serve cluster.outbound statistics via 15000/stats
        # in addition to the stats normally served by Istio.  The Circuit Breaking example task
        # gives an example of inspecting Envoy stats.
        sidecar.istio.io/statsInclusionPrefixes: cluster.outbound,cluster_manager,listener_manager,http_mixer_filter,tcp_mixer_filter,server,cluster.xds-grpc
      labels:
        app: fortio
    spec:
      containers:
      - name: fortio
        image: fortio/fortio:latest_release
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
          name: http-fortio
        - containerPort: 8079
          name: grpc-ping
          

```
 
 ```
 kubectl apply -f httpbin_fortio.yaml
 ```
 
i.  Log in to the client pod and use the fortio tool to call httpbin. Pass in curl to indicate that you just want to make one call
 
```
export FORTIO_POD=$(kubectl get pods -lapp=fortio -o 'jsonpath={.items[0].metadata.name}')
kubectl exec "$FORTIO_POD" -c fortio -- /usr/bin/fortio curl -quiet http://httpbin:8000/get
```

j. the request succeeded! Now, break something.

k. Call the service with two concurrent connections (-c 2) and send 20 requests (-n 20):

```
kubectl exec "$FORTIO_POD" -c fortio -- /usr/bin/fortio load -c 2 -qps 0 -n 20 -loglevel Warning http://httpbin:8000/get
```

l. Bring the number of concurrent connections up to 3:

```
kubectl exec "$FORTIO_POD" -c fortio -- /usr/bin/fortio load -c 3 -qps 0 -n 30 -loglevel Warning http://httpbin:8000/get
```

m. Query the istio-proxy stats to see more
```
kubectl exec "$FORTIO_POD" -c istio-proxy -- pilot-agent request GET stats | grep httpbin | grep pending
```

n. You can see the upstream_rq_pending_overflow value which means calls so far have been flagged for circuit breaking.

o. Delete all the applied files for circuit-breaking.

26. Retries

a. When a service tries to reach another svc and for some reason it fails, we can configure virtual svc to attempt that operation again

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
    - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
    retires:
      attempts: <No of attempts>
        perTryTimeout: <Interval between retries>
 ```
 
 27. Service authentication
 
a. Within ISTIOD there is a CA that manages keys and certificates in Istio

b. Envoy proxy req the certificate

c. With mutual TLS one service is authenticated with another svc

d. Certification generation and distribution is managed by Istiod

```
apiVersion: 
kind: Peerauthentication
metadata:
  name:
  namespace: <On which namespace u want to apply>
spec:
  selector: 
    matchLabels: <Workload specific for one svc , if removed then the policy is for entire namespace>
  mtls:
    mode: STRICT
```

28. RBAC

a. Istio RBAC (Role Based Access Control) defines ServiceRole and ServiceRoleBinding objects.

b. A ServiceRole specification includes a list of rules (permissions). Each rule has the following standard fields:

i. services: a list of services.

ii. methods: HTTP methods. In the case of gRPC, this field is ignored because the value is always “POST”.

iii. paths: HTTP paths or gRPC methods. Note that gRPC methods should be presented in the form of “/packageName.serviceName/methodName” and are case sensitive.

c. servicerole.yaml

```
apiVersion: "rbac.istio.io/v1alpha1"
kind: ServiceRole
metadata:
  name: products-viewer
  namespace: default
spec:
  rules:
  - services: ["products.svc.cluster.local"]
    methods: ["GET", "HEAD"]
    constraints:
    - key: "destination.labels[version]"
      values: ["v1", "v2"]
```

d. The above service-role has “read” (“GET” and “HEAD”) access to “products.svc.cluster.local” service at versions “v1” and “v2”. “path” is not specified, so it applies to any path in the service.

e. ServiceRoleBinding specification includes two parts:

i. The roleRef field that refers to a ServiceRole object in the same namespace.

ii.A list of subjects that are assigned the roles.

f. A simple user field, operators can also use custom keys in the properties field, the supported keys are listed in the “constraints and properties” page.

g. servicerolebinding.yaml

```
apiVersion: "rbac.istio.io/v1alpha1"
kind: ServiceRoleBinding
metadata:
  name: test-binding-products
  namespace: default
spec:
  subjects:
  - user: alice@yahoo.com
  - properties:
      source.namespace: "abc"
  roleRef:
    kind: ServiceRole
    name: "products-viewer"
<It is binding the two subject to a role product-viewers>
```

29. Telemetry

a. Telemetry defines how the metrics is generated for workloads within a mesh.

b. The hierarchy of Telemetry configuration is as follows:

i. Workload-specific configuration

ii. Namespace-specific configuration

iii. Root namespace configuration

c. Policy to enable random sampling for 10% of traffic:

```
apiVersion: telemetry.istio.io/v1alpha1
kind: Telemetry
metadata:
  name: mesh-default
  namespace: istio-system
spec:
  # no selector specified, applies to all workloads
  tracing:
  - randomSamplingPercentage: 10.00
  - disableSpanReporting: true <For disabling>
    customTags:      <Pilicy to add custom tags>
      my_new_foo_tag:
        literal:   
          value: "foo"
          
  metrics:    <To disable server-side metrics for Stackdriver for an entire mesh>
  - providers:
    - name: stackdriver
    overrides:
    - match:
        metric: ALL_METRICS
        mode: SERVER
      disabled: true
      
   accessLogging: <To enable access logging of entire mesh >
  - providers:
    - name: envoy
  - disabled: true <For disabling >
```

d. Telemetry is measured by:

i. selector: The selector decides where to apply the Telemetry policy. If not set, the Telemetry policy will be applied to all workloads in the same namespace as the Telemetry policy.

ii. tracing: Tracing configures the tracing behavior for all selected workloads.

iii. metrics: Metrics configure the metrics behavior for all selected workloads.

iv. accessLogging: AccessLogging configures the access logging behavior for all selected workloads.






















