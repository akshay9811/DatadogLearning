Question 1: Configure Logging on the Datadog Agent

Task:

    Enable logging on the Datadog Agent.

    Configure a custom log file at /var/log/custom_app.log.

    Set up the Agent to collect logs with service:custom_app, source:python, and env:prod.

    Restart the Agent and verify logs are being collected.

✅ Solution:

Step 1: Enable Logging in Main Config

Edit datadog.yaml:

sudo nano /etc/datadog-agent/datadog.yaml

Enable log collection:

logs_enabled: true

Step 2: Create Log Configuration

Create a log config:

sudo mkdir -p /etc/datadog-agent/conf.d/custom_app.d
sudo nano /etc/datadog-agent/conf.d/custom_app.d/conf.yaml

Paste:

logs:
  - type: file
    path: /var/log/custom_app.log
    service: custom_app
    source: python
    env: prod

Step 3: Create Dummy Log File

sudo touch /var/log/custom_app.log
sudo chmod 666 /var/log/custom_app.log

Step 4: Restart Agent

sudo systemctl restart datadog-agent

Step 5: Verify

sudo datadog-agent status

Look for the Logs Agent section and custom_app.
✅ Question 2: Configure Network Performance Monitoring (NPM)

Task:

    Enable Network Performance Monitoring in the Agent.

    Configure it to tag flows with env:test, team:infra.

    Restart and verify in the Agent status output.

✅ Solution:

Step 1: Enable NPM in main config

Edit datadog.yaml:

network_config:
  enabled: true
tags:
  - env:test
  - team:infra

Step 2: Kernel Dependency (Ubuntu)

sudo apt-get install -y linux-headers-$(uname -r)

Step 3: Restart

sudo systemctl restart datadog-agent

Step 4: Verify

sudo datadog-agent status

Check for network performance monitoring section.
✅ Question 3: Collect System Metrics from a Docker Container

Task:

    Run a container named mycontainer.

    Enable container integration in Datadog.

    Collect CPU and memory metrics for mycontainer.

    Tag it with env:docker-lab.

✅ Solution:

Step 1: Run Docker Container

docker run -d --name mycontainer nginx

Step 2: Enable Docker Integration

Edit datadog.yaml:

listeners:
  - name: docker

config_providers:
  - name: docker
    polling: true

tags:
  - env:docker-lab

Step 3: Restart Agent

sudo systemctl restart datadog-agent

Step 4: Verify

sudo datadog-agent status




