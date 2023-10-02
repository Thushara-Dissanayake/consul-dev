# ****************** Manual Canary Releases with Consul (Without a Release Controller) ********************

k port-forward svc/ui 8080:80
    http://localhost:8080/ui/

Test Command:

while true; do curl -s curl -s http://app-lb-233976837.eu-central-1.elb.amazonaws.com | jq ' .upstream_calls."http://web.default:9090".name'; sleep 1; done


-------------------------------------------------------------

nicholas jackson's fakse service and consul release controller:
    https://github.com/nicholasjackson/fake-service
    https://nicholasjackson.io/consul-release-controller/example_app/