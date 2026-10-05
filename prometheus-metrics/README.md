# Prometheus Metrics Exporter Extension

**Version:** 1.1.0

A Prometheus metrics exporter for DeskStride that enables workflow monitoring and observability by exposing a `/metrics` endpoint for Prometheus scraping.

## Features

- **Record Counter**: Track monotonically increasing values (events, requests, errors)
- **Record Gauge**: Track values that can go up or down (temperature, queue size, memory usage)
- **Record Histogram**: Track value distributions (request durations, response sizes)
- **HTTP Metrics Endpoint**: Expose Prometheus-compatible `/metrics` endpoint for scraping

## Setup

### Extension Settings

Configure in the Extensions page:

- **Metrics Prefix**: Prefix for all metric names (e.g., `deskstride_workflow` -> `deskstride_workflow_runs_total`)
- **Enable Default Metrics**: Include Python process metrics (CPU, memory, GC, etc.) in `/metrics` endpoint

### HTTP Routes

This extension provides an HTTP route that must be enabled:

1. Go to Settings → Server (Local API)
2. Enable the local API server
3. On the Extensions card, enable **Allow HTTP routes**
4. The `/metrics` endpoint will be available at:
   ```
   http://127.0.0.1:8765/ext/prometheus-metrics/metrics
   ```

### Prometheus Configuration

Add this to your `prometheus.yml`:

```yaml
scrape_configs:
  - job_name: 'deskstride'
    static_configs:
      - targets: ['localhost:8765']
    metrics_path: '/ext/prometheus-metrics/metrics'
    bearer_token: 'your-api-key'  # From Settings → API
```

## Nodes

### Record Counter
Record a counter metric (monotonically increasing value). Use for counting events, requests, errors, etc.

**Inputs:**
- Metric Name (required): Name of the counter metric (e.g., `api_requests_total`)
- Increment Value: Amount to increment by (default: 1)
- Labels: Label key=value pairs, comma-separated (e.g., `status=200,endpoint=/api`)
- Description: Human-readable description of the metric

**Outputs:**
- success: Boolean indicating if metric was recorded
- metricName: The metric name
- value: The increment value
- labels: The labels applied
- message: Status message

### Record Gauge
Record a gauge metric (can go up or down). Use for current values like temperature, queue size, memory usage.

**Inputs:**
- Metric Name (required): Name of the gauge metric (e.g., `current_users`)
- Value (required): Value to set or increment/decrement by
- Operation: Operation to perform (set, inc, dec)
- Labels: Label key=value pairs, comma-separated (e.g., `server=prod,region=us-east`)
- Description: Human-readable description of the metric

**Outputs:**
- success: Boolean indicating if metric was recorded
- metricName: The metric name
- value: The value used
- operation: The operation performed
- labels: The labels applied
- message: Status message

### Record Histogram
Record a histogram metric (distributions like request durations, response sizes). Use for observing value distributions.

**Inputs:**
- Metric Name (required): Name of the histogram metric (e.g., `request_duration_seconds`)
- Value (required): Value to observe (e.g., duration in seconds)
- Labels: Label key=value pairs, comma-separated (e.g., `endpoint=/api,method=GET`)
- Description: Human-readable description of the metric
- Custom Buckets: Custom bucket boundaries (comma-separated numbers, e.g., `0.1,0.5,1,2,5`)

**Outputs:**
- success: Boolean indicating if metric was recorded
- metricName: The metric name
- value: The observed value
- labels: The labels applied
- message: Status message

## Metric Types

### Counters
Use for things that only increase:
- Number of HTTP requests
- Number of errors
- Number of processed items
- Total workflow runs

### Gauges
Use for things that can go up or down:
- Current memory usage
- Number of active connections
- Queue length
- Temperature

### Histograms
Use for distributions:
- Request duration
- Response size
- Processing time
- File sizes

## Labeling Best Practices

- Use labels to distinguish between different dimensions of the same metric
- Avoid high cardinality labels (e.g., user IDs, timestamps)
- Use consistent label names across metrics
- Keep label values short and meaningful

**Good examples:**
- `status=200,endpoint=/api`
- `server=prod,region=us-east`
- `method=GET,content_type=json`

**Bad examples:**
- `user_id=12345` (high cardinality)
- `timestamp=2023-09-17T10:30:00Z` (high cardinality)
- `very_long_label_name_with_much_detail=value` (too verbose)

## Use Cases

- **Workflow Monitoring**: Track workflow execution counts, durations, and success rates
- **API Monitoring**: Monitor request counts, response times, error rates
- **Resource Monitoring**: Track memory usage, CPU usage, disk space
- **Business Metrics**: Track orders processed, revenue generated, user signups
- **Alerting**: Set up Prometheus alerts based on metric thresholds

## Example Workflows

### Track Workflow Success Rate
```
1. Start Workflow
2. Try:
   - Execute main workflow logic
   - Record Counter: metricName=workflow_success, labels=workflow_name=process_orders
3. Except:
   - Record Counter: metricName=workflow_error, labels=workflow_name=process_orders
4. End
```

### Monitor API Response Times
```
1. Start Timer
2. HTTP Request to API
3. Calculate Duration
4. Record Histogram: metricName=api_duration_seconds, value=duration, labels=endpoint=/api
```

### Track Queue Size
```
1. Get Queue Length
2. Record Gauge: metricName=queue_size, value=length, operation=set, labels=queue=orders
```

## Requirements

- prometheus-client==0.19.0 (included in extension)
- Local API server enabled in DeskStride
- HTTP routes allowed for extensions
- Prometheus server (optional, for scraping metrics)

## Notes

- Metrics are stored in memory and reset when the extension is unloaded
- Use labels carefully to avoid high cardinality metrics
- The `/metrics` endpoint requires the local API to be enabled
- Prometheus scraping requires the API key for authentication
- Default Python metrics are optional and can be enabled in settings
- Histograms provide automatic bucketing for percentile calculations

## Security

- The `/metrics` endpoint is protected by the same API key authentication as other local API endpoints
- Configure Prometheus with the appropriate `bearer_token` in the scrape config
- Ensure your API key is kept secure and rotated regularly
- Consider firewall rules to restrict access to the metrics endpoint

## Contract (v1.1.0)

**Install:** Extensions → **Install from file** → choose `prometheus-metrics.dsext`.

**Permissions:**
- **httpRoutes**

No example workflow: recording a counter alone needs the HTTP metrics route enabled to observe output.

**Lint / scan / pack:**
```bash
python tools/deskstride_ext_cli.py check extensions/prometheus-metrics
python tools/deskstride_ext_cli.py pack extensions/prometheus-metrics -o prometheus-metrics.dsext
```
