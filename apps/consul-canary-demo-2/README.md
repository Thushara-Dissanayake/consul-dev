# Automated Canary Releases with Consul using ArgoRollout + Gateway API 

1. Install ArgoRollout

2. Mounting the API GW plugin executable into the rollouts controller container using config map
    k apply -f configmap.yaml

        Argo API GW controller plugin:
        https://github.com/argoproj-labs/rollouts-plugin-trafficrouter-gatewayapi/tags

    ** You may need to restart rollout controller pods after this 

3. Enable API GW in Consul
    It will do followings automatically, 
        a. Create Gateway entrypoint
        b. Create GatewayClass and Gateway resources
        c. Create cluster entrypoint and map it with our Gateway entrypoint

4. Create HTTPRoute
    k apply -f httproute.yaml

5. Create cluster role for rollout controller to acces API GW and HTTP route resources
    k apply -f clusterrole.yaml

6. Bind above role to rollout controller SA
    k apply -f rollbinding.yaml

7. Create canary and stable services
    k apply -f services.yaml

8. Configure trafficRouting for gatewayAPI in 

      trafficRouting:
        plugins:
          argoproj-labs/gatewayAPI:
            httpRoute: route-canary-demo-2
            namespace: default

9. Create intentions from API GW to rollout-service

10. Create argo-rollouts resources
    k apply -f rollout.yaml

11. Monitor
    k argo rollouts get rollout rollouts-demo --watch




