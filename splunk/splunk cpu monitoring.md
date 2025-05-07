## Splunk: Monitoring CPU Usage - A Concise Explanation

Splunk can monitor CPU usage through various methods, primarily by ingesting performance data. Here's a breakdown:

### 1. Data Collection Methods:

* **Operating System Performance Monitoring Tools:**
    * **Windows:** Perfmon (Performance Monitor)
    * **Linux/Unix:** `sar`, `vmstat`, `top`, `/proc` filesystem
* **Agent-Based Collection (Splunk Universal Forwarder):**
    * The Universal Forwarder is installed on the target machine.
    * It collects performance logs and metrics.
    * **Configuration:** You configure the Forwarder to monitor specific performance counters or log files related to CPU usage.
* **Agentless Collection (e.g., via APIs or scripts):**
    * Splunk can ingest data from APIs that expose system metrics.
    * Custom scripts can be written to gather CPU information and send it to Splunk.

### 2. Key Data Points for CPU Monitoring:

* **CPU Utilization (%):** The percentage of time the CPU is actively processing tasks.
* **System Time (%):** The percentage of time the CPU spends running kernel-level code.
* **User Time (%):** The percentage of time the CPU spends running user-level code.
* **Idle Time (%):** The percentage of time the CPU is idle.
* **Wait Time (%):** The percentage of time the CPU is waiting for I/O operations to complete.
* **Interrupt Time (%):** The percentage of time the CPU spends handling hardware interrupts.
* **Context Switches:** The number of times the operating system switches between different processes.
* **CPU Load Average (Linux/Unix):** Represents the average number of processes in the run queue over a specific period (1, 5, and 15 minutes).

### 3. Splunk Configuration Examples (Illustrative):

* **`inputs.conf` (on the Forwarder for Perfmon - Windows):**
    ```ini
    [perfmon://CPU]
    object = Processor
    counters = % Processor Time;% User Time;% Privileged Time;% Idle Time
    instances = _Total
    interval = 10
    index = perfmon
```
* **`inputs.conf` (on the Forwarder for `sar` output - Linux):**
```ini
    [monitor:///var/log/sa/sa*]
    sourcetype = sysstat_sar
    index = linux_metrics
```
*(Assuming `sar` logs are being collected)*

### 4. Splunk Search Queries for Analysis:

* **Average Overall CPU Utilization:**
```ini
    index=perfmon sourcetype="Perfmon:CPU" counter="% Processor Time" instance="_Total"
    | timechart avg(_value) AS "Average CPU Utilization (%)"
```
* **CPU Utilization by Core (if available):**
```ini
    index=perfmon sourcetype="Perfmon:CPU" counter="% Processor Time"
    | timechart avg(_value) BY instance
```
* **High CPU Usage Detection (Thresholding):**
```ini
    index=perfmon sourcetype="Perfmon:CPU" counter="% Processor Time" instance="_Total"
    | where avg(_value) > 80
    | timechart avg(_value)
    | alert
```
* **Analyzing `sar` data (Linux):**
```ini
    index=linux_metrics sourcetype=sysstat_sar
    | search CPU=*
    | timechart avg(CPU.*)
```

### 5. Visualization and Alerting:

* Splunk allows you to create dashboards and visualizations (e.g., line charts, area charts, single value panels) to monitor CPU trends in real-time.
* You can set up alerts to trigger notifications when CPU usage exceeds predefined thresholds, indicating potential performance issues.

In essence, Splunk acts as a central platform to collect, index, and analyze CPU performance data from various sources, enabling you to gain insights into system resource utilization and troubleshoot performance bottlenecks.

---
## Splunk Universal Forwarder & CPU Usage: A Brief Example

The **Splunk Universal Forwarder** is a lightweight agent that you install on machines you want to monitor. It's **not agentless**; it requires installation on each system. Its primary job is to collect data and securely forward it to your Splunk Indexer.

### How it Collects CPU Usage:

The Forwarder relies on the underlying operating system's tools to gather CPU metrics. Here's a simplified view:

1.  **Configuration:** You tell the Forwarder *what* data to collect and *how* often. This is done through configuration files (primarily `inputs.conf`).
2.  **Data Acquisition:** Based on your configuration, the Forwarder uses OS-specific mechanisms:
    * **Windows:** It queries Performance Monitor (Perfmon) for CPU-related counters like "% Processor Time".
    * **Linux/Unix:** It can monitor the output of commands like `sar`, `vmstat`, or directly read files in the `/proc` filesystem that contain CPU statistics.
3.  **Data Forwarding:** The Forwarder packages this raw CPU data and securely transmits it to your Splunk Indexer.

### Simple Setup Example (Illustrative):

Let's say you want to monitor overall CPU usage (%) on a Windows machine.

1.  **Install the Universal Forwarder:** Download and install the Forwarder on the Windows machine. During installation, you'll be prompted for the IP address or hostname of your Splunk Indexer.

2.  **Configure `inputs.conf`:** On the Windows machine, navigate to the Forwarder's configuration directory (e.g., `C:\Program Files\SplunkUniversalForwarder\etc\system\local`). Create or edit the `inputs.conf` file with the following content:

   ```ini
    [perfmon://CPU]
    object = Processor
    counters = % Processor Time
    instances = _Total
    interval = 60
    index = windows_metrics
```

* `[perfmon://CPU]`: Defines a stanza for monitoring Perfmon counters related to "CPU".
* `object = Processor`: Specifies the Perfmon object to monitor.
	* `counters = % Processor Time`: Tells Splunk to collect the "% Processor Time" counter (overall CPU utilization). You could add more counters separated by semicolons.
	* `instances = _Total`: Monitors the aggregate CPU usage across all cores. To monitor individual cores, you might use `instances = *`.
	* `interval = 60`: Specifies that data should be collected every 60 seconds.
	* `index = windows_metrics`: Defines the Splunk index where this data will be stored.

3.  **Restart the Forwarder:** After saving `inputs.conf`, restart the Splunk Universal Forwarder service on the Windows machine for the changes to take effect.

Now, the Forwarder will start collecting the overall CPU utilization every minute and send it to your Splunk Indexer, where you can search and analyze the `windows_metrics` index.

**Key takeaway:** The Universal Forwarder acts as an agent, actively gathering data based on your configuration and forwarding it to the central Splunk instance. It's not agentless as it requires software to be installed on the monitored system.

## Splunk Universal Forwarder on Linux & CPU Usage: A Brief Example

Since you're on a Linux machine, the Universal Forwarder will leverage different tools to collect CPU metrics. Remember, it's **not agentless**.

### How it Collects CPU Usage (Linux):

Similar to Windows, the Forwarder relies on Linux system utilities and files:

1.  **Configuration:** You define what CPU data to collect and how often in the Forwarder's configuration files (`inputs.conf`).
2.  **Data Acquisition:** The Forwarder can be configured to:
    * **Monitor `/proc/stat`:** This virtual file provides detailed kernel statistics, including CPU usage broken down by various categories (user, system, idle, etc.).
    * **Execute commands:** You can configure it to periodically run commands like `sar`, `vmstat`, or `top` and ingest their output.
    * **Monitor log files:** Some applications or system services might log CPU-related information.
3.  **Data Forwarding:** The collected CPU data is packaged and securely sent to your Splunk Indexer.

### Simple Setup Example (Illustrative - Monitoring `/proc/stat`):

Here's a basic example of configuring the Forwarder to monitor the `/proc/stat` file for CPU usage:

1.  **Install the Universal Forwarder:** Download and install the Forwarder on your Linux machine. During installation, provide the IP address or hostname of your Splunk Indexer.

2.  **Configure `inputs.conf`:** Navigate to the Forwarder's configuration directory (e.g., `/opt/splunkforwarder/etc/system/local`). Create or edit the `inputs.conf` file with the following content:

```ini
    [monitor:///proc/stat]
    sourcetype = proc_stat
    index = linux_metrics
```

* `[monitor:///proc/stat]`: Defines a stanza to monitor the `/proc/stat` file. The `monitor://` input monitors a file and reads new lines as they are added.
* `sourcetype = proc_stat`: Assigns a sourcetype to the data, making it easier to parse and analyze in Splunk.
* `index = linux_metrics`: Specifies the Splunk index where this data will be stored.

3.  **Create a Props and Transforms Configuration (for better parsing):** To easily extract CPU fields from the raw `/proc/stat` data, you'll typically create or edit `props.conf` and `transforms.conf` in the same directory (`/opt/splunkforwarder/etc/system/local`).

    * **`props.conf`:**
	
```
[proc_stat]
SHOULD_LINEMERGE = false
EXTRACT-proc_stat_fields = ^cpu\s+(?P<user>\d+)\s+(?P<nice>\d+)\s+(?P<system>\d+)\s+(?P<idle>\d+)\s+(?P<iowait>\d+)\s+(?P<irq>\d+)\s+(?P<softirq>\d+)(?:\s+(?P<steal>\d+))?(?:\s+(?P<guest>\d+))?(?:\s+(?P<guest_nice>\d+))?	
```

- `SHOULD_LINEMERGE = false`: Prevents Splunk from trying to merge multiple lines into a single event.
-  `EXTRACT-proc_stat_fields`: Defines a regular expression to extract CPU statistics into named fields.

- **`transforms.conf`:**
			*(This file might not be strictly necessary for basic extraction but is often used for more complex transformations. In this simple case, `props.conf` handles the extraction.)*

4.  **Restart the Forwarder:** After saving the configuration files, restart the Splunk Universal Forwarder service on your Linux machine.

Now, the Forwarder will read the `/proc/stat` file periodically, and Splunk will index the data with the `sourcetype` `proc_stat`. You can then use Splunk search queries to calculate CPU utilization based on the extracted fields (e.g., `user`, `system`, `idle`).

**Key takeaway:** On Linux, the Universal Forwarder can monitor system files like `/proc/stat` to gather raw CPU statistics. You might need to define extractions in `props.conf` to make the data easily searchable and usable in Splunk. It is not an agentless solution.