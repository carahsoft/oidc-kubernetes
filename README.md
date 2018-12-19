# How to Secure your Kubernetes Cluster with OIDC

- RBAC
  - Grant permission to admin and developer users
  - Developer / admin users will be created their own namespace with full access
  - Developer users will have full access to shared namespaces and view access to the cluster
  - Admin users will have full access to the cluster
- ODIC Proxy Dashboard
  - OpenResty NGINX resverse proxy to secure the Kubernetes Dashboard
- Kuberos
  - Provides a web interface to download a config file for kubectl

## ADFS Prerequisites
1. Create Application Group 
   - Server applicaiton accessing a web API
2. Server application
   - Make note of Client Identifier
   - Add Redirect URI
     - https://kube-dashboard.yourdomain.com/oauth2/callback
     - https://kube-ctl.yourdomain.com/ui
3. Configure Application Credentials
   - Check Generate a shared secret
4. Configure Web API
   - Add Client Identifier from step 2 to Identiifer list
5. Configure Application Permissions
   - Check email, openid, profile

## Kubernetes Prerequisites
1. Install Kubernetes Dashboard
   - https://github.com/kubernetes/dashboard
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```
2. Configure Kubernetes API Servers
   - Issuer URL is OpenID Connect without /.well-known/openid-configuration
   - vim /etc/kubernetes/manifests/kube-apiserver.yaml
```
     - --oidc-issuer-url=FILL IN YOUR ISSUER URL FOR ADFS
     - --oidc-client-id=FILL IN YOUR CLIENT IDENTIFIER FROM ADFS
     - --oidc-username-claim=upn
     - --oidc-username-prefix=-
```
   - Restart kublet service
```
systemctl restart kubelet
```
3. Make sure you have an ingress controller
   - https://kubernetes.github.io/ingress-nginx/deploy/

## Installation
1. Download kontemplate
   - https://github.com/tazjin/kontemplate/releases
2. Clone a copy of this repo
3. Import SSL certificates for kube-dashboard.yourdomain.com (oidc-proxy-dashboard) and kube-ctl.yourdomain.com (kuberos)
```
kubectl create secret tls kubernetes-dashboard-ingress-certs --key yourdomain.com.key --cert yourdomain.com.crt -n kube-system
kubectl create secret tls kubernetes-ctl-ingress-certs --key yourdomain.com.key --cert yourdomain.com.crt -n kube-system
```
3. Edit cluster.yaml for your environment
4. Generate, review, and apply configuration 
```
kontemplate template -i rbac cluster.yaml > rbac.yaml
kontemplate template -i oidc-proxy-dashboard cluster.yaml > oidc-proxy-dashboard.yaml
kontemplate template -i kuberos cluster.yaml > kuberos.yaml
kubectl apply -f rbac.yaml
kubectl apply -f oidc-proxy-dashboard.yaml
kubectl apply -f kuberos.yaml
```
