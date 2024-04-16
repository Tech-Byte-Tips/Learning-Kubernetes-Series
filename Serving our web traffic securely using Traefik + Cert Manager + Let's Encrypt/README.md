# Please Support This Project!

I would appreciate a donation if you found it useful.

[![](https://www.paypalobjects.com/en_US/i/btn/btn_donateCC_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_donations&business=53CD2WNX3698E&lc=US&item_name=TechByteTips&item_number=Learning%2dKubernetes%2dSeries&currency_code=USD&bn=PP%2dDonationsBF%3abtn_donateCC_LG%2egif%3aNonHosted)

You can also support me by sending a BitCoin donation to the following address:

19JXFGfRUV4NedS5tBGfJhkfRrN2EQtxVo

# Serving our web traffic securely using Traefik + Cert Manager + Let's Encrypt

In this tutorial we are using the k3s built-in ingress controller, Traefik, as the ingress controller for our web services.  Additionally, we are going to use Cert-Manager to request and retrieve Certificates from the Let's Encrypt Certification Authority.  This will make sure that all the web traffic that we serve is secured, reliable, and trustworthy for our users.

## Why use Cert-Manager?

Using Cert-Manager in Kubernetes simplifies the SSL/TLS certificate management process as it automates the acquisition, renewal, and deletion of the certificates.  It integrates seamlessly with Kubernetes resources and provides flexibility to work with various certificate issuers.  This helps us to increase the security of our applications while reducing the operational overhead.

## Notes

For this series, I will not be using Helm.  I will be using plain manifest files.  That way, we can see what we are doing in details.  Applying someone's Helm chart is convenient but, you should get used to figuring out what you are applying to your cluster, YAML manifest files are the way to do this.

This guide assumes that you have already exposed your ingress controller through your router.  Let's Encrypt will not be able to reach your Kubernetes cluster to validate your domain using the HTTP ACME method if you don't do this.  You will need to port forward ports 80 and 443 to your cluster.

---

# Steps

1. Clone this repository

    ```
    git clone https://github.com/Tech-Byte-Tips/Learning-Kubernetes-Series/
    ```

2. [Install k3s](https://github.com/Tech-Byte-Tips/Reference-Guides/blob/main/K3S-In-Netbooted-Raspberry-Pis.md) (if you don't already have it).  This guide assumes that you use k3s.

3. By default, k3s comes with Traefik pre-configured as its ingress controller.  If you are using a different version of Kubernetes, you might need to install it:

    ```
    kubectl apply -f Traefik-CertManager-LetsEncrypt/01 - Traefik
    ```

4. Install Cert Manager:

    ```
    kubectl apply -f "Traefik-CertManager-LetsEncrypt/02 - Cert Manager/cert-manager-1.13.1.yaml"
    ```
    or 
    ```
    kubectl apply -f "https://github.com/cert-manager/cert-manager/releases/download/v1.13.1/cert-manager.yaml"
    ```

    You can download the YAML to install Cert Manager from their [releases page](https://github.com/cert-manager/cert-manager/releases/) in GitHub.  As of writing, the latest version is 1.13.1.

5. Validate that Cert Manager was properly installed:

    ```
    kubectl get pods -n cert-manager
    ```

    This should show you a list of 3 pods for Cert Manager.  These are the CA Injector, the Webhook, and the Cert Manager instance.

6. Create a Cluster Certificate Issuer entry for Let's Encrypt:

    ```
    kubectl apply -f "Traefik-CertManager-LetsEncrypt/02 - Cert Manager/le-staging-issuer.yaml
    ```
    or
    ```
    kubectl apply -f "Traefik-CertManager-LetsEncrypt/02 - Cert Manager/le-production-issuer.yaml
    ```

    Here we are defining two Certificate Issuers for Let's Encrypt.  The Staging one is for testing, as we don't want to hit their rate limit while testing.  Once you are sure that it is working properly, you can switch to the Production Issuer.

7. Verify that the issuer was properly applied:

    ```
    kubectl get ClusterIssuer -A
    ```

    It should return an entry with the status of Ready = True if everything went well.

8. Check the status of the Cluster Issuer by describing it:

    ```
    kubectl describe clusterissuer letsencrypt-staging
    ```
    or 
    ```
    kubectl describe clusterissuer letsencrypt-production
    ```

    If you can see Status = True, everything is working properly.

9. For testing, we will deploy a default nginx image:

    ```
    kubectl create deployment nginx --image nginx:alpine
    ```

10. Check the status of the deployment:

    ```
    kubectl get deployments
    ```

    ```
    kubectl describe deployment nginx
    ```

11. Expose it through port 80:

    ```
    kubectl expose deployment nginx --port 80 --target-port 80
    ```

12. Check that the service is running:

    ```
    kubectl get services
    ```

13. Create an ingress traefik controller:

```
# ingress ingress-nginx.yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-production
    kubernetes.io/ingress.class: traefik
  labels:
    app: nginx
  name: nginx
  namespace: default
spec:
  rules:
  - host: example.com # Change by your domain
    http:
      paths:
      - backend:
          service:
            name: nginx
            port: 
              number: 80
        path: /
        pathType: Prefix  
  tls:
  - hosts:
    - example.com # Change by your domain
    secretName: example-com-tls
```

    ```
    kubectl apply -f ingress-nginx.yaml
    ```

14. Test that it works by navigating to the website in your browser.