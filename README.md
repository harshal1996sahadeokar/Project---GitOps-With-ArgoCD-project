# ArgoCD on Minikube Quickstart

This guide provides a concise walkthrough to set up ArgoCD on Minikube and deploy a sample application using GitOps.

## Prerequisites

- Minikube installed and running
- kubectl installed
- git installed
- GitHub account

## Steps

1.  **Start Minikube:**

    ```bash
    minikube start --driver=docker
    kubectl cluster-info
    ```

2.  **Install ArgoCD:**

    ```bash
    kubectl create namespace argocd
    kubectl apply -n argocd -f [https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml]
    kubectl get pods -n argocd -w # Wait for pods to be Running
    ```

3.  **Access ArgoCD Dashboard:**

    ```bash
    kubectl port-forward svc/argocd-server -n argocd 8080:443
    ```

    Open `https://localhost:8080` in your browser.

4.  **Log in to ArgoCD:**

    ```bash
    kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
    argocd login localhost:8080 --username admin --password <password>
    ```

5.  **Deploy Sample Application (GitOps):**

    -   Create a GitHub repository `gitops-argocd-minikube`.
    -   Clone the repository:

        ```bash
        git clone [https://github.com/YOUR_USERNAME/gitops-argocd-minikube.git](https://github.com/YOUR_USERNAME/gitops-argocd-minikube.git)
        cd gitops-argocd-minikube
        mkdir k8s && cd k8s
        ```

    -   Create `deployment.yaml`:

        ```yaml
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: my-app
          labels:
            app: my-app
        spec:
          replicas: 2
          selector:
            matchLabels:
              app: my-app
          template:
            metadata:
              labels:
                app: my-app
            spec:
              containers:
                - name: my-app
                  image: nginx
                  ports:
                    - containerPort: 80
        ```

    -   Create `service.yaml`:

        ```yaml
        apiVersion: v1
        kind: Service
        metadata:
          name: my-app-service
        spec:
          selector:
            app: my-app
          ports:
            - protocol: TCP
              port: 80
              targetPort: 80
          type: NodePort
        ```

    -   Commit and push:

        ```bash
        git add .
        git commit -m "Added Kubernetes manifests"
        git push origin main
        ```

6.  **Create ArgoCD Application:**

    ```bash
    argocd app create my-app \
      --repo [https://github.com/YOUR_USERNAME/gitops-argocd-minikube.git](https://github.com/YOUR_USERNAME/gitops-argocd-minikube.git) \
      --path k8s \
      --dest-server [https://kubernetes.default.svc](https://kubernetes.default.svc) \
      --dest-namespace default
    argocd app sync my-app
    ```

7.  **Verify Deployment:**

    ```bash
    kubectl get pods
    kubectl get svc
    minikube service my-app-service
    ```

8.  **Cleanup (Optional):**

    ```bash
    argocd app delete my-app
    kubectl delete namespace argocd
    minikube stop
    ```
