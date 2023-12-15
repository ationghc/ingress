# ingress

`Enable ingress:`
1. minikube addons enable ingress

`Confirm nginx controller pods are running:`
2. kubectl get pods -n ingress-nginx

`Apply the following YAML to create an ingress for the vault-ui service:`
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: vault-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: "vault.hashicorp.com"
    http:
      paths:
      - path: /ui
        pathType: Prefix
        backend:
          service:
            name: vault-ui
            port:
              number: 8200
```
3. kubectl apply -f ingress-vault-ui.yaml -n vault

`Confirm ingress IP address:`
4. kubectl get ingress -n vault -o yaml

`Create dns “A” record for vault.hashicorp.com and point it to your Ingress controller IP.`
5. kubectl edit configmap coredns -n kube-system
```
hosts {
           192.168.65.2 host.minikube.internal
           Add_Your_Ingress_IP vault.hashicorp.com
           fallthrough
        }
```
`Scale down coredns deployment to 0 running pods`
6. kubectl scale deploy coredns --replicas=0 -n kube-system

`Scale up coredns deployment to 1 running pod`
7. kubectl scale deploy coredns --replicas=1 -n kube-system

`Ensure your coredns pod is running`
8. kubectl get po -n kube-system -l k8s-app=kube-dns

`If the coredns pod isn’t running, then check logs for syntax error`
9. kubectl logs coredns-your_hash -n kube-system

`Exec into pod and perform a dns lookup on vault.hashicorp.com. Ensure it points to your ingress Address`

10. kubectl exec -it vault-0 -n vault -- /bin/sh
11. Nslookup vault.hashicorp.com

`Create an nginx deployment and expose deployment as a service`
12. kubectl create deploy nginx --image nginx --port=80 -n vault
13. kubectl expose deployment nginx --type=NodePort -n vault

`Use the following YAML to create an ingress for the nginx service:`
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-service
spec:
  ingressClassName: nginx
  rules:
  - host: "vault.hashicorp.com"
    http:
      paths:
      - path: /nginx-test
        pathType: Prefix
        backend:
          service:
            name: nginx
            port:
              number: 80
```
14. kubectl apply -f ingress-nginx.yaml -n vault


`Exec into vault-0 and validate both ingresses are routing request to the correct service.`
15. kubectl exec -it vault-0 -n vault -- /bin/sh
16. curl -v http://vault.hashicorp.com/nginx-test
17. curl -v http://vault.hashicorp.com/ui/login
