# vibe-media-deploy
Deployments for Vibe Media app

## Deploying Locally

Follow these steps to set up and deploy the Vibe Media app locally using Kubernetes and Argo CD.

1. **Create a Kind Cluster**
    Create a local Kubernetes cluster using Kind with the specified configuration for ingress:
    ```bash
    kind create cluster --config ingress-local//kind-ingress.yaml
    ```
2. **Install NGINX Ingress Controller**
    Apply the NGINX ingress controller manifest for Kind:
    ```bash
    kubectl apply -f \
    https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
    ```
    Wait until you see the pod is Running:
    ```
    kubectl get pods -n ingress-nginx
    ```

3. **Install Argo CD**
    Create a namespace for Argo CD:
    ```bash
    kubectl create namespace argocd
    ```
    Install Argo CD in the argocd namespace:

    ```bash
    kubectl apply -n argocd \
    -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    ```

4. **Configure Argo CD Ingress**
    Apply the Argo CD ingress configuration to expose the Argo CD UI:
    ```bash
    kubectl apply -f ingress-local/argocd-ingress.yaml
    ```

5. **Retrieve Argo CD Admin Password**
    Retrieve the initial admin password for Argo CD:
    ```bash
    export ARGOCD_ADMIN_PASSWORD="$(kubectl -n argocd \
    get secret argocd-initial-admin-secret \
    -o jsonpath='{.data.password}' | base64 -d)"
    ```
    Now you can visit localhost:800 and your ArgoCD app will be visible.
    Log in with the following:
    - **Username**: admin
    - **Password**: take from step 5

6. **Creating the Vibe-Media App**
    Log into argo via CLI
    ```bash
    argocd login localhost:8080 --username admin --password $ARGOCD_ADMIN_ASSWORD
    ```
    Create Argo APP for vibe-media
    ```bash
    argocd app create vibe-media \
        --repo https://github.com/jameshyphen/vibe-media-deploy \
        --path apps/vibe-media \
        --revision HEAD \
        --dest-server https://kubernetes.default.svc \
        --dest-namespace default \
        --project default \
        --sync-policy automated \
        --self-heal \
        --auto-prune
    ```
7. **Add vibe.local to /etc/hosts**
    Edit the file at /etc/hosts and add the following line:
    ```bash
    127.0.0.1       vibe.local
    ```

Done. You can now visit vibe.local on your browser.