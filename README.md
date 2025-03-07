# Project : ArgoCD on Minikube Quickstart

This guide provides a concise walkthrough to set up ArgoCD on Minikube and deploy a sample application using GitOps.
![image](https://github.com/user-attachments/assets/e7eb8cb6-4ee2-40af-829a-8d740556c9b0)


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
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    kubectl get pods -n argocd -w # Wait for pods to be Running
    ```
    ![image](https://github.com/user-attachments/assets/82142152-1b04-4f0e-85e2-570879edece4)
    ![image](https://github.com/user-attachments/assets/24734d07-b9bd-45b0-ac45-6a54baa58bab)



4.  **Access ArgoCD Dashboard:**

    ```bash
    kubectl port-forward svc/argocd-server -n argocd 8081:443
    ```

    Open `https://localhost:8081` in your browser.
    ![image](https://github.com/user-attachments/assets/9887b828-1bed-4235-a1b9-2ee0ca524ede)


6.  **Log in to ArgoCD:**

    ```bash
    kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
    argocd login localhost:8081 --username admin --password <password>
    ```
    ![image](https://github.com/user-attachments/assets/ab4f30d5-ca24-4afb-b94a-3549c5400b00)


7.  **Deploy Sample Application (GitOps):**

    -   Create a GitHub repository `Project---GitOps-With-ArgoCD-project`.
    -   Clone the repository:

        ```bash
        git clone https://github.com/harshal1996sahadeokar/Project---GitOps-With-ArgoCD-project.git
        #cd gitops-argocd-minikube
        #mkdir k8s && cd k8s
        ```

    -   Create `deployment.yaml`:


    -   Create `service.yaml`:

    -   Commit and push:

        ```bash
        git add .
        git commit -m "Added Kubernetes manifests"
        git push origin main
        ```

8.  **Create ArgoCD Application:**

    ```bash
    argocd app create my-app \
      --repo https://github.com/harshal1996sahadeokar/Project---GitOps-With-ArgoCD-project.git \
      --path k8s \
      --dest-server [https://kubernetes.default.svc](https://kubernetes.default.svc) \
      --dest-namespace default
    argocd app sync my-app
    ```

9.  **Verify Deployment:**

    ```bash
    kubectl get pods
    kubectl get svc
    minikube service my-app-service
    ```

10.  **Cleanup (Optional):**

    ```bash
    argocd app delete my-app
    kubectl delete namespace argocd
    minikube stop
    ```
