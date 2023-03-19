# Week 2 — Distributed Tracing

### Goal: 
Implement distributed tracing as it will become difficult to pinpoint issue and fix them once a good number of cloud services are involved in the project which also helps in keepng pace in development timeline. 

### Technical Tasks: 
1. Instrument the backend flask application to use Open Telemetry (OTEL) with Honeycomb.io as the provider
2. Run queries to explore traces within Honeycomb.io
3. Instrument AWS X-Ray into backend flask application
4. Configure and provision X-Ray daemon within docker-compose and send data back to X-Ray API
5. Observe X-Ray traces within the AWS Console
6. Integrate Rollbar for Error Logging
7. Trigger an error an observe an error with Rollbar
8. Install WatchTower and write a custom logger to send application log data to CloudWatch Log group

--- 

