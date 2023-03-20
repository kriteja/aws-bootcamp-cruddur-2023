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

## 2. Run queries to explore traces within Honeycomb.io

- :heavy_check_mark: Honeycomb received the trace

![mock data honeycomb](https://user-images.githubusercontent.com/40818088/226205314-814518b0-e570-4765-af98-b343d3e1f1ef.PNG)

- :heavy_check_mark: Honeycomb HeatMap

![Heatmap](https://user-images.githubusercontent.com/40818088/226205540-a1322b34-b147-45b2-a99c-8448e83b09c3.PNG)

## 3. Instrument AWS X-Ray into backend flask application
- Add `aws-xray-sdk` to requirements.txt
- Install Python dependencies 
```bash
pip install -r requirements.txt
```
- Add the below to `app.py`
```py
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.ext.flask.middleware import XRayMiddleware

xray_url = os.getenv("AWS_XRAY_URL")
xray_recorder.configure(service='Cruddur', dynamic_naming=xray_url)
XRayMiddleware(app, xray_recorder)
```
- Setup [AWS X-Ray](https://aws.amazon.com/xray/) resources for sampling data by adding the below to `aws/json/xray.json`
```json
{
  "SamplingRule": {
      "RuleName": "Cruddur",
      "ResourceARN": "*",
      "Priority": 9000,
      "FixedRate": 0.1,
      "ReservoirSize": 5,
      "ServiceName": "Cruddur",
      "ServiceType": "*",
      "Host": "*",
      "HTTPMethod": "*",
      "URLPath": "*",
      "Version": 1
  }
}
```

![Setup X-Ray Resources](https://user-images.githubusercontent.com/40818088/226206052-a6e82b2f-dc4c-4728-8190-ecfe42837185.PNG)

- Establishing connection between AWS X-Ray and IAM User and creating a X-Ray Group
```bash
aws xray create-group \
   --group-name "Cruddur" \
   --filter-expression "service(\"backend-flask\")"
```
- X-Ray Group created message in GitPod terminal

![X-Ray resource created](https://user-images.githubusercontent.com/40818088/226206120-8cbcbc3f-2633-4e83-8819-11c1844fd246.PNG)

- X-Ray Group in AWS Console

![X-Ray resource created in AWS Cloudwatch](https://user-images.githubusercontent.com/40818088/226206152-d1690a32-30f3-4b5f-aeb8-06f0b1b6c3fd.PNG)

- Create a sampling rule with below
```
aws xray create-sampling-rule --cli-input-json file://aws/json/xray.json
```
- Install [X-Ray daemon](https://docs.aws.amazon.com/xray/latest/devguide/xray-daemon.html) to docker-compose.yml
```
  xray-daemon:
    image: "amazon/aws-xray-daemon"
    environment:
      AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
      AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
      AWS_REGION: "us-east-1"
    command:
      - "xray -o -b xray-daemon:2000"
    ports:
      - 2000:2000/udp
```

## 5. Observe X-Ray traces within the AWS Console

- ![Instrument X-Ray Mock-Data](https://user-images.githubusercontent.com/40818088/226206611-1f16fa08-97c5-4a26-b462-c780de3dfcfb.PNG)

## 6. Integrate Rollbar for Error Logging
- Added blinker and rollbar libraries to `requirements.txt`
```
blinker
rollbar
```
- Install the dependencies
```
pip install -r requirements.txt
```
- Set Rollbar token as envirnment variables 
```
export ROLLBAR_ACCESS_TOKEN=""
gp env ROLLBAR_ACCESS_TOKEN=""
```
- Add `ROLLBAR_ACCESS_TOKEN` enviroment variable to backend-flask service in `docker-compose.yml`
```
ROLLBAR_ACCESS_TOKEN: "${ROLLBAR_ACCESS_TOKEN}"
```
- Import Rollbar libraries
```bash
import rollbar
import rollbar.contrib.flask
from flask import got_request_exception
```
- Rollbar listener
```bash
rollbar_access_token = os.getenv('ROLLBAR_ACCESS_TOKEN')
@app.before_first_request
def init_rollbar():
    """init rollbar module"""
    rollbar.init(
        # access token
        rollbar_access_token,
        # environment name
        'production',
        # server root directory, makes tracebacks prettier
        root=os.path.dirname(os.path.realpath(__file__)),
        # flask already sets up logging
        allow_logging_basic_config=False)

    # send exceptions from `app` to rollbar, using flask's signal system.
    got_request_exception.connect(rollbar.contrib.flask.report_exception, app)
```
- Adding endpoint to test the Rollbar to `app.py`
```bash
@app.route('/rollbar/test')
def rollbar_test():
    rollbar.report_message('Hello World!', 'warning')
    return "Hello World!"
```
- :heavy_check_mark: Hello World trace in Rollbar 

![Rollbar Hello World](https://user-images.githubusercontent.com/40818088/226293166-21c137ee-9d65-4aa9-93db-2157791f479e.PNG)

- :heavy_check_mark: Hello World output 

![Rollbar test](https://user-images.githubusercontent.com/40818088/226293351-9f09ae09-9973-4467-b1d9-6945b251472b.PNG)

## 7. Trigger an error an observe an error with Rollbar

![Rollbar error](https://user-images.githubusercontent.com/40818088/226293531-271a75ca-b560-4d51-8e0c-e47c6bd71409.PNG)

## 8. Install WatchTower and write a custom logger to send application log data to CloudWatch Log group
- Setup watchtower lib in backend-flask `requirements.txt`
```
watchtower
```
- Install the dependencies 
```
pip install -r requirements.txt
```
- Add the following code to `app.py`
```py
import watchtower
import logging
from time import strftim
```
```py
# Configuring Logger to Use CloudWatch
LOGGER = logging.getLogger(__name__)
LOGGER.setLevel(logging.DEBUG)
console_handler = logging.StreamHandler()
cw_handler = watchtower.CloudWatchLogHandler(log_group='cruddur')
LOGGER.addHandler(console_handler)
LOGGER.addHandler(cw_handler)
LOGGER.info("some message")
```
```py
@app.after_request
def after_request(response):
    timestamp = strftime('[%Y-%b-%d %H:%M]')
    LOGGER.error('%s %s %s %s %s %s', timestamp, request.remote_addr, request.method, request.scheme, request.full_path, response.status)
    return response
```





















