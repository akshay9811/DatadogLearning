**Advanced Datadog Troubleshooting Lab Guide using Amazon EC2**

---

### **Preparation: Creating the Environment and Installing the Datadog Agent**

#### **1. Launch Amazon EC2 Instances**

* Go to the AWS EC2 Dashboard and launch 2–3 t2.micro or t3.small instances (Ubuntu preferred).
* Open ports 22 (SSH), 80 (HTTP), and 8125 (Datadog statsd default) in the security group.

#### **2. Simulate a Service or Application**

* SSH into the instance:

  ```bash
  ssh -i <your-key>.pem ubuntu@<your-ec2-public-ip>
  ```
* Clone a sample web app or service:

  ```bash
  git clone https://github.com/DataDog/sample-app.git
  cd sample-app
  ```
* Run it with Docker or Python/Node.js depending on the app.

#### **3. Install the Datadog Agent**

* Create an API key from your Datadog dashboard.
* On the EC2 instance, run:

  ```bash
  DD_API_KEY=<your_api_key> bash -c "$(curl -L https://s3.amazonaws.com/dd-agent/scripts/install_script.sh)"
  ```
* Confirm installation:

  ```bash
  sudo datadog-agent status
  ```
* Enable APM and Logs in `/etc/datadog-agent/datadog.yaml`:

  ```yaml
  apm_config:
    enabled: true

  logs_enabled: true
  ```
* Restart the agent:

  ```bash
  sudo systemctl restart datadog-agent
  ```

#### **4. Optional Integrations**

* Enable integrations like NGINX, PostgreSQL, Kafka by creating config files in `/etc/datadog-agent/conf.d/<integration>/conf.yaml`.

---

### **Exercise 1: Microservice Timeout in Production During High Load**

**How to Create the Issue**:

* Run a basic Python or Node.js service that calls another endpoint with artificial delay (e.g., `sleep(3)`).
* Use `ab` or `wrk` to simulate load: `ab -n 1000 -c 20 http://localhost:5000/`.

**Solution**:

1. Go to **APM > Services > orders-api**.
2. Filter traces by duration (e.g., > 1s) and status code (e.g., 500).
3. Identify the longest trace and view its flame graph.
4. Use trace tags (like `host`, `availability-zone`) to correlate the timeout to specific EC2 instances.
5. Open **Infrastructure > EC2 host** and check metrics: CPU, memory, network I/O.
6. Verify if auto-scaling group added instances. If not, adjust policies.
7. Check if downstream service (e.g., database) is the real bottleneck by viewing service map.

---

### **Exercise 2: Intermittent Errors After Canary Deployment**

**How to Create the Issue**:

* Deploy two versions of a service: one stable, one with a bug that returns 500 errors.
* Use tags: `DD_VERSION=stable` and `DD_VERSION=canary`.

**Solution**:

1. Use **Log Explorer** and filter logs by tag `version:<canary>` and `status_code:500`.
2. Compare logs from `version:stable` and `version:canary`.
3. Go to **APM > Services > \[app]** and use the `version` tag to compare performance.
4. On **Dashboards**, compare request latency and error rate grouped by version.
5. Use `host` tags to find which EC2 instance(s) are running the canary and restart or roll back if needed.

---

### **Exercise 3: Memory Leak in Python Worker**

**How to Create the Issue**:

* Run a Python script that appends large data to a list in a loop.

**Solution**:

1. Go to **Infrastructure > Host > \[instance]** and select memory metrics.
2. Use **Live Processes** to identify the Python process with rising memory.
3. Go to **Logs > \[service]** and search for `MemoryError`, `Killed`, or OOM traces.
4. Enable Python profiler (`DD_PROFILING_ENABLED=true`) and check memory profiling in APM.
5. Create a custom monitor to alert if memory exceeds threshold (e.g., 80%).

---

### **Exercise 4: Log Volume Spike**

**How to Create the Issue**:

* Enable a debug logger in a tight loop.

**Solution**:

1. Go to **Logs > Analytics** and group by `host` and `service`.
2. Identify log source with spike (e.g., a verbose debug logger).
3. Filter logs by `log_level:debug` or `log_level:info`.
4. Review and fix logging configuration on the offending EC2 instance.
5. Use **Log Exclusion Filters** in Datadog to drop unnecessary logs.
6. Set a **Log Volume Monitor** to alert on spikes in log size.

---

### **Exercise 5: DNS Failures in EC2-hosted Services**

**How to Create the Issue**:

* Modify `/etc/resolv.conf` to an invalid nameserver or simulate network block.

**Solution**:

1. Enable **Network Performance Monitoring** on Datadog agent.
2. Go to **Network > DNS** and filter by `host` or `service`.
3. Look for spikes in `dns.failure_rate` or increased `latency`.
4. Correlate with **Logs**: search `connection timed out` or `NXDOMAIN`.
5. Check `/etc/resolv.conf` on EC2 instance and restart DNS-related services if needed.

---

### **Exercise 6: Latency Spikes in ap-southeast-1 Region**

**How to Create the Issue**:

* Use a geo-aware application or simulate slow external API call in that region.

**Solution**:

1. Go to **APM > \[Service] > Endpoint** and filter by `region:ap-southeast-1`.
2. Compare latency trends across regions.
3. Check **Infrastructure > EC2 Hosts** in that region for CPU/network spikes.
4. Investigate if RDS or external service calls are slower in that region.
5. Use **Synthetic Monitors** to simulate requests from that region.

---

### **Exercise 7: Kafka Consumer Lag**

**How to Create the Issue**:

* Deploy Kafka on EC2, produce large message volumes with slow consumers.

**Solution**:

1. Use Datadog Kafka integration and view `kafka.consumer_lag` per topic.
2. Go to **Infrastructure > EC2** and verify CPU/Memory of the consumer instance.
3. Look at APM traces (if enabled) to view processing time per message.
4. Check for GC metrics, throttling, or I/O saturation.
5. Add custom metrics like `messages_processed_per_second` to track throughput.

---

### **Exercise 8: AWS RDS Connection Throttling**

**How to Create the Issue**:

* Set low `max_connections` in RDS and create many simultaneous app connections.

**Solution**:

1. Use **AWS Integration > RDS** dashboard and view `DatabaseConnections`, `CPUUtilization`, `FreeableMemory`.
2. Correlate connection spikes with **APM traces** or EC2 instance tags (e.g., deployment, host).
3. Verify if connection pooling is used and properly configured in application code.
4. Create a dashboard to compare `connection count` across services/instances.
5. Set alert if RDS connections approach max limit.

---

This lab guide can be used for hands-on troubleshooting simulations on Amazon EC2 environments instrumented with the Datadog Agent.
