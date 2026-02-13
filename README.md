Multi-Store WordPress + WooCommerce on Kubernetes



Overview



This project demonstrates a Kubernetes-based store provisioning system that deploys independent WordPress/WooCommerce stores with namespace isolation. Built as a learning exercise starting from zero Kubernetes knowledge.



Key Capability: Deploy multiple isolated e-commerce stores on a single Kubernetes cluster, each with its own database, configuration, and domain access.



What This System Does



\- Runs multiple independent WooCommerce stores on local Kubernetes (KIND)

\- Each store operates in its own namespace for isolation

\- Persistent storage ensures data survives pod restarts

\- Domain-based routing via Kubernetes Ingress (locally exposed using port-forward due to KIND limitations)

\- Proven end-to-end order placement capability



Architecture



Traffic Flow:



Browser

&nbsp; → Ingress Controller (NGINX)

&nbsp; → Service (ClusterIP)

&nbsp; → WordPress Pod (Deployment)

&nbsp; → MySQL Service

&nbsp; → MySQL Pod (StatefulSet + PVC)



Design Decisions:



\- WordPress: Deployed as a Deployment (stateless application layer)

\- MySQL: Deployed as a StatefulSet with PersistentVolumeClaim (stateful data layer)

\- Isolation: Each store runs in its own namespace

\- Networking: Services for internal communication, Ingress for external routing

\- Storage: PVCs ensure database persistence across pod lifecycles



Technologies Used



\- Docker Desktop - Container runtime

\- Kubernetes (KIND) - Local Kubernetes cluster

\- kubectl - Kubernetes CLI

\- NGINX Ingress Controller - HTTP routing

\- WordPress 6.9.x - CMS/application layer

\- WooCommerce - E-commerce functionality

\- MySQL 8.0 - Database



Kubernetes Resources Used



\- Namespace

\- Deployment

\- StatefulSet

\- Service (ClusterIP)

\- Ingress

\- PersistentVolumeClaim

\- Pod



Deployed Stores



Store 1: Namespace store1, Domain store1.local

Store 2: Namespace store2, Domain store2.local



Isolation Proof: Both stores run simultaneously. Deleting one namespace does not affect the other.



Project Structure



wordpress-k8s-stores/

├── mysql.yaml

├── mysql-store2.yaml

├── wordpress.yaml

├── wordpress-store2.yaml

├── wordpress-ingress.yaml

├── wordpress-ingress-store2.yaml

├── screenshots/

│   ├── order-placed.png

│   ├── orders-admin.png

│   ├── pods-running.png

│   └── ingress-routing.png

└── README.md



Setup Instructions



Prerequisites:

\- Docker Desktop installed and running

\- kubectl CLI installed

\- KIND installed



1\. Create Kubernetes Cluster



kind create cluster --name store-cluster



2\. Install NGINX Ingress Controller



kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml



kubectl wait --namespace ingress-nginx --for=condition=ready pod --selector=app.kubernetes.io/component=controller --timeout=90s



3\. Create Namespaces



kubectl create namespace store1

kubectl create namespace store2



4\. Deploy MySQL (StatefulSet + PVC)



kubectl apply -f mysql.yaml -n store1

kubectl apply -f mysql-store2.yaml -n store2



5\. Deploy WordPress



kubectl apply -f wordpress.yaml -n store1

kubectl apply -f wordpress-store2.yaml -n store2



6\. Create Services



kubectl expose deployment wordpress --name=wordpress-svc --port=80 -n store1

kubectl expose deployment wordpress --name=wordpress-svc --port=80 -n store2



7\. Configure Ingress



kubectl apply -f wordpress-ingress.yaml

kubectl apply -f wordpress-ingress-store2.yaml



8\. Update Local DNS



Edit /etc/hosts (Mac/Linux) or C:\\Windows\\System32\\drivers\\etc\\hosts (Windows):



127.0.0.1 store1.local

127.0.0.1 store2.local



9\. Expose Ingress Locally



Important: KIND runs locally and doesn't have cloud LoadBalancers. Expose Ingress via port-forward:



kubectl port-forward -n ingress-nginx svc/ingress-nginx-controller 80:80



Keep this terminal running while accessing stores.



10\. Access Stores



Store 1: http://store1.local

Store 2: http://store2.local



Testing End-to-End Order Flow



Complete WordPress Setup:

1\. Visit http://store1.local

2\. Complete WordPress installation wizard

3\. Install WooCommerce plugin (Plugins → Add New → WooCommerce)

4\. Run WooCommerce setup wizard

5\. Enable "Cash on Delivery" payment (WooCommerce → Settings → Payments)



Create and Test Product:

1\. Go to Products → Add New

2\. Create a test product with price

3\. Publish product

4\. Visit store homepage

5\. Add product to cart

6\. Proceed to checkout

7\. Fill dummy customer details

8\. Select "Cash on Delivery" payment method

9\. Place order

10\. Verify order appears in WooCommerce → Orders



Screenshots included in /screenshots folder.



What Works



\- Local Kubernetes cluster setup and configuration

\- Multi-store deployment with namespace isolation

\- Persistent MySQL storage (data survives pod restarts)

\- Complete WordPress + WooCommerce installation

\- End-to-end order placement and admin verification

\- Ingress-based domain routing

\- Service-to-service communication (WordPress → MySQL)

\- Repeatable deployment process



Known Limitations



What Was Not Implemented:



\- No Helm Charts - Used raw YAML manifests instead

\- No Dashboard UI - Manual kubectl commands for management

\- No Production Configs - Local development setup only

\- No Advanced Multi-Tenant Security - Basic namespace isolation only

\- No Monitoring/Logging - No Prometheus, Grafana, or centralized logging

\- No Automation Scripts - No create-store.sh or delete-store.sh

\- No Secret Management - Passwords hardcoded in YAML (not production-safe)

\- Local Ingress Exposure - Requires port-forward due to KIND limitations



Conscious Tradeoffs:



These limitations were conscious decisions based on:

\- Time constraints (5 days from zero Kubernetes knowledge)

\- Focus on understanding core concepts over tooling

\- Prioritizing a working, explainable system

\-While no automation scripts were created, the deployment process is fully repeatable using documented kubectl commands.



Production Readiness Gap



What Would Be Required for Production:



Infrastructure:

\- Cloud Kubernetes cluster (GKE, EKS, AKS)

\- Real LoadBalancer for Ingress

\- Managed MySQL or Cloud SQL

\- Storage classes for dynamic provisioning



Security:

\- Kubernetes Secrets for credentials

\- RBAC policies

\- Network policies

\- TLS/HTTPS with cert-manager

\- Container image scanning



Automation:

\- Helm charts for templating

\- Kustomize for environment-specific configs

\- CI/CD pipeline (GitOps)



Observability:

\- Prometheus + Grafana for monitoring

\- Centralized logging (ELK/Loki)

\- Distributed tracing



Scaling:

\- Horizontal Pod Autoscaling

\- Resource limits and quotas

\- Multi-node cluster with node affinity



Key Learnings



Technical Concepts Learned:

\- Kubernetes Architecture: Control plane, nodes, pods, controllers

\- Stateful vs Stateless Workloads: When to use Deployments vs StatefulSets

\- Persistent Storage: PVCs, storage classes, volume lifecycle

\- Networking: Services (ClusterIP, NodePort, LoadBalancer), Ingress controllers

\- Namespace Isolation: Resource boundaries and multi-tenancy basics

\- Pod Lifecycle: Self-healing, restart policies, readiness/liveness



Practical Skills Gained:

\- Using kubectl for cluster management

\- Writing Kubernetes YAML manifests

\- Debugging pod failures (describe, logs, exec)

\- Understanding local vs cloud Kubernetes differences

\- Deploying real applications (not just nginx examples)



What I'd Do Differently:

\- Start with Helm charts from beginning

\- Use Kubernetes Secrets from the start

\- Implement proper logging earlier

\- Test deletion/cleanup workflows earlier



Troubleshooting



Common Issues:



Pods stuck in Pending state:

kubectl describe pod <pod-name> -n <namespace>

Check Events section for errors



Can't access store via domain:

\- Verify /etc/hosts entry exists

\- Ensure port-forward is running

\- Check Ingress rules: kubectl get ingress -A



Database connection errors:

\- Verify MySQL pod is running: kubectl get pods -n <namespace>

\- Check MySQL logs: kubectl logs <mysql-pod> -n <namespace>



Order data not persisting:

\- Verify PVC is bound: kubectl get pvc -n <namespace>

\- Check PV status: kubectl get pv



Verification Commands



View all namespaces:

kubectl get namespaces



View all resources in store1:

kubectl get all -n store1



Check Ingress configuration:

kubectl get ingress -A



Verify persistent volumes:

kubectl get pvc -A



Check pod logs:

kubectl logs <pod-name> -n <namespace>



Test database connection:

kubectl exec -it <mysql-pod> -n store1 -- mysql -uroot -p<password>



Cleanup



To remove a store completely:

kubectl delete namespace store1



This removes all resources (pods, services, PVCs) in that namespace.



To delete the entire cluster:

kind delete cluster --name store-cluster



Conclusion



This project demonstrates:



\- Learning Ability - Went from zero Kubernetes knowledge to deploying multi-store systems in 5 days

\- Understanding Over Completeness - Built less, understood more

\- Honest Engineering Assessment - Clear about what works and what doesn't

\- Real Working System - Not theoretical - proven end-to-end order flow



While this implementation doesn't include production-grade tooling (Helm, monitoring, advanced security), it represents a solid foundation demonstrating Kubernetes fundamentals and the ability to learn complex distributed systems quickly.

f

The gap between this implementation and production readiness is acknowledged and documented, showing realistic understanding of what enterprise-grade deployments require.



Author



Built as part of Urumi AI SDE Internship technical assessment (Round 1)



Timeline: February 8-13, 2026 (5 days)



Starting Knowledge: Zero Kubernetes/DevOps experience

