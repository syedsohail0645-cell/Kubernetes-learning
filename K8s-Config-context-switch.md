I have been using multiple Kubernetes clusters in my organization, and I often need to switch between different clusters, such as development and production.

To manage this efficiently, the following steps are used.

Steps to Switch Between Kubernetes Clusters and Namespaces

(i) To check how many Kubernetes configurations (contexts) are available and to see which one is currently active, use the command below:

kubectl config get-contexts


(ii) To switch the Kubernetes configuration to a different cluster, such as a production or development cluster, use the following command:

kubectl config use-context prod-cluster


Replace prod-cluster with the required context name based on your environment.

(iii) To change the current namespace from the default namespace to the required namespace, use the command below:

kubectl config set-context --current --namespace=kube-system


This command updates the namespace for the currently active context.

(iv) To check and confirm which Kubernetes configuration (context) is currently in use, run the command below:

kubectl config current-context
