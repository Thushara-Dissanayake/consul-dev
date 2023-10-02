################### ArgoRollout Without Service Mesh ###################

Arogo Rollout controller and Kubernetes plugin installation 

    1. Install Argo Rollout
    
    https://argo-rollouts.readthedocs.io/en/release-1.5/installation/
    
    or

    Using Helm:
    helm repo add argo https://argoproj.github.io/argo-helm && helm repo update argo
        - https://github.com/argoproj/argo-helm/blob/main/charts/argo-rollouts/values.yaml

    helm upgrade argo-rollouts argo/argo-rollouts --version 2.31.2 -f values.yaml --create-namespace --namespace argo-rollouts --install

    2. Attach IAM policy to EKS node group IAM role to Get metric data from cloudwatch (If Cloudwatch is used in Analysis template)

    3. Set AWS_REGION variable in argorollout deployment  (If Cloudwatch is used in Analysis template)

        apiVersion: apps/v1
        kind: Deployment
        metadata:
        name: argo-rollouts
        spec:
        template:
            spec:
            containers:
            - name: argo-rollouts
              env:
                - name: AWS_REGION
                  value: eu-central-1

    4. k apply -f rollouts-demo-ingress.yaml -n test

    5. k apply -f analysis-template.yaml -n test
        * update the load balancer value in AnalysisTemplate first

    6. k apply -f services.yaml -n test


Canary deployment

    https://argo-rollouts.readthedocs.io/en/stable/getting-started/

    k create ns test

    a. Deploying a Rollout

        k apply -f rollout.yaml -n test

        kubectl  argo rollouts get rollout rollouts-demo --watch -n test

    b. Updating a Rollout

        kubectl argo rollouts set image rollouts-demo rollouts-demo=argoproj/rollouts-demo:yellow -n test

    c. Promoting a Rollout

        kubectl argo rollouts promote rollouts-demo -n test

    d. Aborting a Rollout

        kubectl argo rollouts abort rollouts-demo -n test

-----

k argo rollouts list rollouts

k argo rollouts dashboard

k get analysisrun

Rollout Specification: https://argoproj.github.io/argo-rollouts/features/specification/