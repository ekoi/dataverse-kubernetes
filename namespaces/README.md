
        kubectl get namespaces --show-labels

        kubectl apply -f namespace-minikube.json
        kubectl get namespaces --show-labels
        kubectl config set-context local-dev --namespace=local-dev --cluster=minikube --user=minikube


        kubectl config view
result:
 
        - context:
            cluster: minikube
            namespace: local-dev
            user: minikube
          name: local-dev

kubectl config current-context
kubectl config use-context local-dev