# Istio Setup
Install Istio 
```
curl -L https://git.io/getLatestIstio | sh -
cp istio-1.0.5/bin/istioctl /usr/local/bin/
cd istio-1.0.5/
kubectl apply -f install/kubernetes/helm/istio/templates/crds.yaml
kubectl apply -f install/kubernetes/istio-demo-auth.yaml
```
Confirm istio namespace resources being created (istio-system)
```
watch kubectl get all --all-namespaces
```
Enable istio injection in default namespace
```
kubectl label namespace default istio-injection=enabled
kubectl get ns default -o yaml
```
Istio Ingress
* Determining the ingress IP and ports
```
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')
export INGRESS_HOST=$(kubectl get po -l istio=ingressgateway -n istio-system -o 'jsonpath={.items[0].status.hostIP}')
export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
```
# Bookinfo sample
```
cd istio-1.0.5/
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
kubectl apply -f  samples/bookinfo/networking/bookinfo-gateway.yaml
kubectl apply -f samples/bookinfo/networking/destination-rule-all-mtls.yaml
curl -o /dev/null -s -w "%{http_code}\n" http://${GATEWAY_URL}/productpage
curl -so - http://${GATEWAY_URL}/productpage
```
* Istio Routing
```
istioctl get destinationrules
kubectl create -f samples/bookinfo/networking/virtual-service-all-v1.yaml
kubectl get virtualservice -o yaml
kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-50-v3.yaml
kubectl get virtualservice -o yaml
```
Istio Gateway
```
cat <<EOF | kubectl apply -f -
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: httpbin-gateway
spec:
  selector:
    istio: ingressgateway # use Istio default gateway implementation
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "httpbin.example.com"
EOF
```
Istio VirtualService
```
cat <<EOF | kubectl apply -f -
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: httpbin
spec:
  hosts:
  - "httpbin.example.com"
  gateways:
  - httpbin-gateway
  http:
  - match:
    - uri:
        prefix: /status
    - uri:
        prefix: /delay
    route:
    - destination:
        port:
          number: 8000
        host: httpbin
EOF
```
Httpbin Service and Deployment
```
kubectl apply -f httpbin.yaml
```
Test
```
curl -I -HHost:httpbin.example.com http://$INGRESS_HOST:$INGRESS_PORT/status/200
curl -I -HHost:httpbin.example.com http://$INGRESS_HOST:$INGRESS_PORT/headers
```

```
cat <<EOF | kubectl apply -f -
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: httpbin-gateway
spec:
  selector:
    istio: ingressgateway # use Istio default gateway implementation
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
  name: httpbin
spec:
  hosts:
  - "*"
  gateways:
  - httpbin-gateway
  http:
  - match:
    - uri:
        prefix: /headers
    route:
    - destination:
        port:
          number: 8000
        host: httpbin
EOF
```
http://$INGRESS_HOST:$INGRESS_PORT/headers

* Bookinfo sample delete
```
kubectl delete -f samples/bookinfo/platform/kube/bookinfo.yaml
// kubectl delete -f install/kubernetes/istio-demo-auth.yaml
kubectl delete gateway httpbin-gateway
kubectl delete virtualservice httpbin
kubectl delete --ignore-not-found=true -f samples/httpbin/httpbin.yaml
```