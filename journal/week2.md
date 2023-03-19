# Week 2 — Distributed Tracing

## Goal: 
Implement distributed tracing as it will become difficult to pinpoint issue and fix them once a good number of cloud services are involved in the project which also helps in keepng pace in development timeline. 

## Technical Tasks: 
1. Instrument the backend flask application to use Open Telemetry (OTEL) with Honeycomb.io as the provider
2. Run queries to explore traces within Honeycomb.io
3. Instrument AWS X-Ray into backend flask application
4. Configure and provision X-Ray daemon within docker-compose and send data back to X-Ray API
5. Observe X-Ray traces within the AWS Console
6. Integrate Rollbar for Error Logging
7. Trigger an error an observe an error with Rollbar
8. Install WatchTower and write a custom logger to send application log data to CloudWatch Log group

--- 

## 1. Instrument the backend flask application to use Open Telemetry (OTEL) with Honeycomb.io as the provider 
### Honeycomb
- Create a free tier account in [Honeycomb.io](https://www.honeycomb.io/)
- Create a envirnment for the Project Cruddur  
- Set Honeycomb API Keys as environment variables using GitPod using the below code
```bash
export HONEYCOMB_API_KEY=""
export HONEYCOMB_SERVICE_NAME="Cruddur"
gp env HONEYCOMB_API_KEY=""
gp env HONEYCOMB_SERVICE_NAME="Cruddur"
```
- Instrument the project application to send Telemetry data to Honeycomb server. *[Honeycomb Documentation](https://docs.honeycomb.io/quickstart/#step-3-instrument-your-application-to-send-telemetry-data-to-honeycomb)*
- Add the following to `requirements.txt`
```
opentelemetry-api 
opentelemetry-sdk 
opentelemetry-exporter-otlp-proto-http 
opentelemetry-instrumentation-flask 
opentelemetry-instrumentation-requests
```
![Open Telementary install](https://user-images.githubusercontent.com/40818088/226205123-2e643259-0c96-4237-984b-d40207321b1e.PNG)

- Install the dependencies using the below command 
```
pip install -r requirements.txt
```
- Add the below lines to `app.py1
```bash
from opentelemetry import trace
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
```
- Add the following environment variables to `backend-flask` in docker compose file.
```bash
OTEL_EXPORTER_OTLP_ENDPOINT: "https://api.honeycomb.io"
OTEL_EXPORTER_OTLP_HEADERS: "x-honeycomb-team=${HONEYCOMB_API_KEY}"
OTEL_SERVICE_NAME: "${HONEYCOMB_SERVICE_NAME}"
```
- :heavy_check_mark: *Honeycomb received the trace*

![mock data honeycomb](https://user-images.githubusercontent.com/40818088/226205314-814518b0-e570-4765-af98-b343d3e1f1ef.PNG)























