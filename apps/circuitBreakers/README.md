k port-forward traffic-generator-7bb84cf465-d7fgm  8080:8080

http://localhost:8080/fortio/

- fake-service has one instance (v2) that returns HTTP 500 error for every request.
- Circuit breaker is configured in ServiceDefaults of traffic-generator
- When the Circuit breaker is triggered, the circuit between traffic gen service and the fake-service instance which returns errors will 'open'.
- After the baseEjectionTime the circuit will be 'closed' again

- Only support 'passive'health checks, which means the circuit will be closed after the defined time even when the error persists. Then, it will be re-opened if error is there.

