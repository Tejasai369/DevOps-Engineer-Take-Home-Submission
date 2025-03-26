#Monitoring Design TXT

from prometheus_client import start_http_server, Gauge
import psutil
import time
import requests

# Check if SSL module is available before making requests
try:
    import ssl
    SSL_AVAILABLE = True
except ModuleNotFoundError:
    SSL_AVAILABLE = False

def ping_latency(url='https://www.google.com'):
    """Measures network latency by sending a request to Google."""
    try:
        response = requests.get(url, timeout=5, verify=SSL_AVAILABLE if SSL_AVAILABLE else False)
        return response.elapsed.total_seconds()
    except requests.exceptions.RequestException:
        return float('inf')  # Return infinity if request fails

# Define Prometheus metrics
CPU_USAGE = Gauge('server_cpu_usage', 'CPU usage percentage')
MEMORY_USAGE = Gauge('server_memory_usage', 'Memory usage percentage')
DISK_USAGE = Gauge('server_disk_usage', 'Disk usage percentage')
NETWORK_LATENCY = Gauge('server_network_latency', 'Network latency in seconds')

def collect_metrics():
    """Collects and updates system metrics."""
    CPU_USAGE.set(psutil.cpu_percent(interval=1))
    MEMORY_USAGE.set(psutil.virtual_memory().percent)
    DISK_USAGE.set(psutil.disk_usage('/').percent)
    NETWORK_LATENCY.set(ping_latency())

if __name__ == "__main__":
    start_http_server(8000)  # Expose metrics on port 8000
    print("Prometheus metrics server started on port 8000")
    while True:
        collect_metrics()
        time.sleep(5)  # Collect metrics every 5 seconds
