# -------------- Prometheus and Grafana Installation with Consul ------------------- #
1. 
Enable metrics under global stanza in in consul helm chart

2. 
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts && \
helm repo update prometheus-community && \
helm repo add grafana https://grafana.github.io/helm-charts && \
helm repo update grafana

3. 
k create ns observability
helm install -f prometheus-values.yaml prometheus prometheus-community/prometheus --version "15.5.3" --create-namespace --namespace observability
or
helm upgrade -f prometheus-values.yaml prometheus prometheus-community/prometheus --version "22.6.2" --create-namespace --namespace observability --install

helm install -f grafana-values.yaml grafana grafana/grafana --version "6.23.1" --create-namespace --namespace observability 
or
helm upgrade -f grafana-values.yaml grafana grafana/grafana --version "6.56.6" --create-namespace --namespace observability --install

4. 
k port-forward -n observability svc/grafana 3000:3000

k port-forward -n observability deploy/prometheus-server 9090:9090
or
k port-forward -n observability svc/prometheus-server 9090:80

5. 
Restart all the app pods to be updated with with propper prometheus annotations

6. 
Login and set Grafana data source to http://prometheus-server.observability.svc.cluster.local
    unsername: admin
    password: kubectl get secret --namespace observability grafana -o jsonpath="{.data.admin-password}" | base64 --decode

7. 
Test:
Prometheus test query: sum by(__name__)({app="product-api"})!= 0

import hashicups-dashboard.json dashboard to grafana and seee the graphs


