# Kubernetes Dashboard OAuth2 SSO
## About
Instructions about how to configure Google OAuth2 SSO authentication in Kubernetes Dashboard.

It's going to use:
- minikube;
- oauth2-proxy;
- Google OAuth2 Application (GCP);

To create a new Google OAuth2 Application clientId and clientSecret, you must follow the instructions:
https://support.google.com/cloud/answer/6158849?hl=en


## Install Minikube
Run the commands:
```
minikube start --driver='docker' --extra-config=apiserver.authorization-mode=Node,RBAC --extra-config=apiserver.oidc-issuer-url=https://accounts.google.com --extra-config=apiserver.oidc-username-claim=email --extra-config=apiserver.oidc-client-id=XXXXXXXXXXXXXXXXXXXXXX.apps.googleusercontent.com
```

## Install NGINX Ingress
Run the commands:
```
minikube addons enable ingress
kubectl patch service ingress-nginx-controller -n ingress-nginx --type='merge' -p '{"spec":{"type":"ClusterIP"}}'
```

## Install Kubernetes Dashboard
Use "minikube ip" and add "192.168.49.2 kubernetes-dashboard.org" entry inside the "/etc/hosts".

Run the commands:
```
minikube addons enable dashboard
kubectl create ingress kubernetes-dashboard -n kubernetes-dashboard --class=nginx --rule="kubernetes-dashboard.org/*=kubernetes-dashboard:80" 
```

### Change dashboard login configuration
Inside kubernetes-dashboard deployment, place the configuration below:

```
args:
  - '--namespace=kubernetes-dashboard'
  - '--enable-skip-login'
  - '--disable-settings-authorizer'
  - '--authentication-mode=bearer'
  - '--enable-insecure-login'
```

### Install oauth2-proxy
Edit the file "oauth2-proxy-values.yaml" and put your Google OAuth2 "clientID", "clientSecret" and "cookieSecret".

To generate the cookie secret, run the commands below:
```
dd if=/dev/urandom bs=32 count=1 2>/dev/null | base64 | tr -d -- '\n' | tr -- '+/' '-_'; echo
```

Run the commands:
```
helm repo add oauth2-proxy https://oauth2-proxy.github.io/manifests
kubectl create namespace oauth2-proxy
kubectl create ingress oauth2-proxy -n oauth2-proxy --class=nginx --rule="kubernetes-dashboard.org/*=oauth2-proxy:80" 
helm upgrade --install oauth2-proxy -n oauth2-proxy oauth2-proxy/oauth2-proxy --values oatuh2-proxy-values.yaml
```

Access the SSO login page:
https://kubernetes-dashboard.org/

* Additional Resources
- Install: https://oauth2-proxy.github.io/oauth2-proxy/docs/
- Configuration: - https://oauth2-proxy.github.io/oauth2-proxy/docs/configuration/overview
- Endpoints: https://oauth2-proxy.github.io/oauth2-proxy/docs/features/endpoints

### Create RBAC resources
Edit the file "rbac.yaml" with your email, etc.

Run the commands:
```
kubectl apply -f rbac.yaml
```
