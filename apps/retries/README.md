
# Retry and Traffic spitting Testing

nw-multitool --> ServiceRouter --> ServiceResolver --> ServiceSplitter --> FakeService v1 & FakeService v2 
or 
nw-multitool --> ServiceRouter --> FakeService

command run from nw-multitool:
    while true; do curl fake-service; sleep 1; done

Observations:
- Works only for HTTP and gRPC (Not tested) and not for TCP
- Both v1 and v2 versions of fake-service returns 500 error for 50% of the requests
- When retry policy configured in serviceRouter, app does not receive 5xx errors.
- Retry logic will be implemented at SOURCE envoy proxy. Therefore, does not work when you directly connect to the service using permissiveMTLS
- Do not work for external virtual services (mariadb) <-- No configs will be added to envoy
