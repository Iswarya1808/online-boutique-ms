<p align="center">
<img src="/src/frontend/static/icons/Hipster_HeroLogoMaroon.svg" width="300" alt="Online Boutique" />
</p>

![Continuous Integration](https://github.com/GoogleCloudPlatform/microservices-demo/workflows/Continuous%20Integration%20-%20Main/Release/badge.svg)

**Online Boutique** is a cloud-first microservices application.
Online Boutique consists of an 11-tier microservices application. The application is a
web-based e-commerce app where users can browse items,
add them to the cart, and purchase them.

This application to demonstrate the use of technologies like
Kubernetes (Azure), Istio, and gRPC. This application
works on any Kubernetes cluster, like Azure
Kubernetes Engine (AKS). It’s **easy to deploy with little to no configuration**.

If you’re using this project, please **★Star** this repository to show your interest!

## Screenshots

| Home Page                                                                                                         | Checkout Screen                                                                                                    |
| ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| [![Screenshot of store homepage](/docs/img/online-boutique-frontend-1.png)](/docs/img/online-boutique-frontend-1.png) | [![Screenshot of checkout screen](/docs/img/online-boutique-frontend-2.png)](/docs/img/online-boutique-frontend-2.png) |

## Quickstart (AKS)

1. Ensure you have the following requirements:
   - [Azure/AWS/Google Cloud account]

2. Clone the repository.

   ```sh
   git clone https://github.com/GoogleCloudPlatform/microservices-demo
   cd microservices-demo/
   ```

5. Deploy Online Boutique to the cluster.

   ```sh
   kubectl apply -f release/kubernetes-manifests.yaml
   ```

6. Wait for the pods to be ready.

   ```sh
   kubectl get pods
   ```

   After a few minutes, you should see the Pods in a `Running` state:

   ```
   NAME                                     READY   STATUS    RESTARTS   AGE
   adservice-76bdd69666-ckc5j               1/1     Running   0          2m58s
   cartservice-66d497c6b7-dp5jr             1/1     Running   0          2m59s
   checkoutservice-666c784bd6-4jd22         1/1     Running   0          3m1s
   currencyservice-5d5d496984-4jmd7         1/1     Running   0          2m59s
   emailservice-667457d9d6-75jcq            1/1     Running   0          3m2s
   frontend-6b8d69b9fb-wjqdg                1/1     Running   0          3m1s
   loadgenerator-665b5cd444-gwqdq           1/1     Running   0          3m
   paymentservice-68596d6dd6-bf6bv          1/1     Running   0          3m
   productcatalogservice-557d474574-888kr   1/1     Running   0          3m
   recommendationservice-69c56b74d4-7z8r5   1/1     Running   0          3m1s
   redis-cart-5f59546cdd-5jnqf              1/1     Running   0          2m58s
   shippingservice-6ccc89f8fd-v686r         1/1     Running   0          2m58s
   ```

7. Access the web frontend in a browser using the frontend's external IP.

   ```sh
   kubectl get service frontend-external | awk '{print $4}'
   ```

   Visit `http://EXTERNAL_IP` in a web browser to access your instance of Online Boutique.

## Exposing app using ingress
   1. Nginx Ingress controller. 
        ```sh
        helm upgrade --install ingress-nginx ingress-nginx \
        --repo https://kubernetes.github.io/ingress-nginx 
        --namespace ingress-nginx --create-namespace
        ```
   2. To see the ingress controller pod running in ingress-nginx namespace. 
        ```sh
        kubectl get pods -n ingress-nginx
        ```
        ```sh
        ingress-nginx-controller-xxxxx   Running   1/1   2m
        ```
  2. Deploying Ingress resource.
       ```sh
       kubectl apply -f release/ingress.yaml
       ```
  3. Access the web frontend in a browser using the Ingress controller external IP.

       ```sh
       kubectl get service -n ingress-nginx | awk '{print $4}'
       ```

   Visit `http://EXTERNAL_IP` in a web browser to access your instance of Online Boutique. 

## Exposing the Application via Istio Gateway and VirtualService

After exposing the app via NGINX Ingress, we configure Istio to manage ingress and egress traffic.
- The Gateway and VirtualServices define how external requests reach the frontend service.
- The ServiceEntries explicitly allow outbound traffic to Google APIs and metadata servers, ensuring secure communication with external dependencies.

   1. The ommand tells Kubernetes to create or update resources defined in the YAML file. These resources are Istio custom objects that extend Kubernetes networking to control how traffic flows inside and outside the mesh. 
        ```sh
        kubectl apply -f release/istio-manifests.yaml
        ```
  2. Verying resources
       ```sh
       kubectl get gateway
       kubectl get virtualservice
       kubectl get serviceentry
       ```

## Architecture

**Online Boutique** is composed of 11 microservices written in different
languages that talk to each other over gRPC.

[![Architecture of
microservices](/docs/img/architecture-diagram.png)](/docs/img/architecture-diagram.png)

## CI/CD Workflow

This project uses GitHub Actions for automated build, quality checks, security scans, and deployment to AKS.

## Pipeline Stages

- **Code Checkout** → triggered by GitHub Actions
- **Quality Gate** → SonarCloud analysis
- **Security Scan** → Trivy vulnerability scan
- **Build & Push** → Docker image pushed to Azure Container Registry (ACR)
- **Deployment** → Helm charts and Istio manifests applied to AKS
- **Post-Deployment** → Smoke tests and monitoring

## Conditional Logic

- If **SonarCloud fails**, the pipeline stops.
- If **Trivy scan fails**, the pipeline stops.
- Only on passing both gates, the image is pushed and deployed.

## Visual Diagram
<img width="1708" height="876" alt="image" src="https://github.com/user-attachments/assets/11bc40a5-5131-4a25-9ecc-2b65fd72f5a6" />


| Service                                              | Language      | Description                                                                                                                       |
| ---------------------------------------------------- | ------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| [frontend](/src/frontend)                           | Go            | Exposes an HTTP server to serve the website. Does not require signup/login and generates session IDs for all users automatically. |
| [cartservice](/src/cartservice)                     | C#            | Stores the items in the user's shopping cart in Redis and retrieves it.                                                           |
| [productcatalogservice](/src/productcatalogservice) | Go            | Provides the list of products from a JSON file and ability to search products and get individual products.                        |
| [currencyservice](/src/currencyservice)             | Node.js       | Converts one money amount to another currency. Uses real values fetched from European Central Bank. It's the highest QPS service. |
| [paymentservice](/src/paymentservice)               | Node.js       | Charges the given credit card info (mock) with the given amount and returns a transaction ID.                                     |
| [shippingservice](/src/shippingservice)             | Go            | Gives shipping cost estimates based on the shopping cart. Ships items to the given address (mock)                                 |
| [emailservice](/src/emailservice)                   | Python        | Sends users an order confirmation email (mock).                                                                                   |
| [checkoutservice](/src/checkoutservice)             | Go            | Retrieves user cart, prepares order and orchestrates the payment, shipping and the email notification.                            |
| [recommendationservice](/src/recommendationservice) | Python        | Recommends other products based on what's given in the cart.                                                                      |
| [adservice](/src/adservice)                         | Java          | Provides text ads based on given context words.                                                                                   |
| [loadgenerator](/src/loadgenerator)                 | Python/Locust | Continuously sends requests imitating realistic user shopping flows to the frontend.                                              |

## Use Terraform to provision a GKE cluster and deploy Online Boutique

The [`/terraform` folder](/terraform) contains instructions for using [Terraform](https://www.terraform.io/intro) to replicate the steps from [**Quickstart (AKS)**](#quickstart-gke) above.
