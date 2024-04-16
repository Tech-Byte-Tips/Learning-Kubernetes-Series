# Please Support This Project!

I would appreciate a donation if you found it useful.

[![](https://www.paypalobjects.com/en_US/i/btn/btn_donateCC_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_donations&business=53CD2WNX3698E&lc=US&item_name=TechByteTips&item_number=Learning%2dKubernetes%2dSeries&currency_code=USD&bn=PP%2dDonationsBF%3abtn_donateCC_LG%2egif%3aNonHosted)

You can also support me by sending a BitCoin donation to the following address:

19JXFGfRUV4NedS5tBGfJhkfRrN2EQtxVo

# Installing ArgoCD

In this tutorial we will be installing ArgoCD, a Continuous Delivery (Continuous Deployment) application that makes our management of the Kubernetes infrastructure a breeze.

## Why use ArgoCD?

It allows us to do GitOps (Git Operations).  This means that the operations of our infrastructure is completely reliant on code in a Git repository.  We define what our infrastructure should look like and ArgoCD will read those definitions.  It will then make sure that the infrastructure in our Kubernetes cluster matches what we described.

---

# Steps

1. Create the DevOps-Tools namespace (if not exists):

  ```
  kubectl create namespace devops-tools
  ```

2. Install ArgoCD in the Kubernetes cluster:

  ```
  kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml -n devops-tools
  ```

3. Get the ArgoCD password:

Linux:

  ```
  kubectl -n devops-tools get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
  ```

Windows:

  ```
  $output = kubectl -n devops-tools get secret argocd-initial-admin-secret -o jsonpath="{.data.password}"

  [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($output))
  ```

4. Expose Argo CD with an ingress and certificate:

  ```
  kubectl apply -f ingress.yaml
  kubectl apply -f configMap.yaml
  ```

  We are applying 2 things:

  a. In here, we are creating an ingress for Traefik pointing to the ArgoCD pod using port 80.

  b. In order to connect using port 80 (terminate the SSL connection by Traefik), we need to make a configuration change to ArgoCD to use HTTP (insecure connections).

5. Log into ArgoCD:

  username: admin
  password: from the step number 3

  NOTE: 

  If you can't see the user interface and get an error related to redirects, you need to restart the ArgoCD server:

  ```
  kubectl rollout restart deployment argocd-server -n devops-tools
  ```

6. Click on Applications and start creating applications.