# Amazon CloudWatch Logs now supports log ingestion using HTTP-based protocol

Posted on: Mar 16, 2026

Amazon CloudWatch Logs now supports HTTP Log Collector (HLC), ND-JSON, Structured JSON and OTEL for sending logs using HTTP-based protocol with bearer token. With this launch, customers can ingest logs where AWS SDK integration is not feasible, such as with third-party or packaged software. The new endpoints are:

- HTTP Log Collector (HLC) Logs (https://logs .<region>.amazonaws.com/services/collector/event) — for JSON events, ideal for migrating existing log pipelines. 
    

- ND-JSON Logs (https://logs.<region>.amazonaws.com/ingest/bulk) — for newline-delimited JSON, where each line is an independent log event. Perfect for high-volume streaming and bulk log ingestion. 
    

- Structured JSON Logs (https://logs .<region>.amazonaws.com/ingest/json) — Send a single JSON object or a JSON array of objects.
    

- OpenTelemetry Logs (https://logs .<region>.amazonaws.com/v1/logs) — for OTLP-formatted logs in JSON or Protobuf encoding to CloudWatch.
    

To enable the HLC endpoint, navigate to CloudWatch Settings in the AWS Console and generate an API key. CloudWatch creates the necessary IAM user with service-specific credentials and permissions. API keys can be configured with expiration periods of 1, 5, 30, 90, or 365 days. Customers must enable bearer token authentication on each log group before it can accept logs, which protects from unintended ingestion. Customers can use service control policies to block the creation of service-specific credentials.

These endpoints are available in the following AWS Regions: US East (N. Virginia), US West (N. California), US West (Oregon), and US East (Ohio). To learn more about the HLC endpoint and security best practices, refer to the [CloudWatch Logs Documentation](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CWL_HTTP_Endpoints.html).