# Questions:
-  what is the purpose of Splunk
	1. how does it work? 
		- explain the whole purpose
		- what is the difference between it and (Prometheus & Grafana)
	2. what is the difference between metric & stats?
	3. difference between the commands `tstats` `stats` `mstats`
	4. how to show percent of CPU usage in our pod
	5. what is a lookup and do you use it?
	6. what is the base search?
	7. what is the difference between:
	  verbose mode, smart mode, fast mode?
	  what mode should we work with for creating filters at ease

# Answers:
#####  How does Splunk works
-  Splunk Process:
	- Data Collection: Servers, Applications, Networks devices and IoT Devices will have Data for Splunk to collect (logs, metrics events machine data)
	- Index: Splunk will index the data to make it searchable. involves parsing data and storing it in structured format
	- Search and Query: Then Users Can Query the Structured Data
		- SPL is the splunk search processing language
		- this will let us extract insight with complex queries
	- Visualization: There are Dashboards, graphs and reports. 
	- Alerting: splunk generates alerts based on defined conditions and automate Reponses to certain events
- Difference between splunk and Prometheus

| Feature         | Prometheus                   | Splunk                                            |
| --------------- | ---------------------------- | ------------------------------------------------- |
| Purpose         | monitoring time series data  | log manage security analytics                     |
| Data Collection | pull-based(scraping metrics) | push-based ( collects logs from multiple sources) |
| Query Language  | PromQL                       | SPL                                               |
| Visualization   | grafana, Highly customizable | Built in                                          |
##### Metric vs Events

| Aspect          | Metrics                                     | Events                                                           |
| --------------- | ------------------------------------------- | ---------------------------------------------------------------- |
| Definition      | Num measurements collected in regular times | Discrete occurrences or actions happening at irregular intervals |
| Purpose         | Tracks Trends over Time                     | Specific moments / changes                                       |
| Characteristics | evenly Spaced data                          | unpredictable data                                               |
| Example         | average response time every minute          | user click event or a server crush                               |
##### `tstats` `stats` `mstats`

| **Command** | **Purpose**                                                       | **Data Scope**                                                         | **Performance**                                                | **Use Case**                                                                |
| ----------- | ----------------------------------------------------------------- | ---------------------------------------------------------------------- | -------------------------------------------------------------- | --------------------------------------------------------------------------- |
| **tstats**  | Performs statistical queries on indexed fields in **tsidx** files | Works on **index-time fields** (e.g., sourcetype, host, source, _time) | **Fastest**, as it uses pre-indexed metadata                   | Ideal for accelerated data models and metadata-based searches               |
| **stats**   | Calculates aggregate statistics on raw event data                 | Works on **search-time fields** (all fields in raw events)             | Slower than tstats, as it processes raw events                 | General-purpose statistical analysis on raw data                            |
| **mstats**  | Performs statistical queries on **metric data**                   | Works on **metric indexes** (optimized for time-series data)           | Optimized for metric data, faster than stats for this use case | Best for analyzing time-series metrics like CPU usage or memory consumption |
##### Show CPU usage:


##### Lookup


##### Base Search


##### Modes



----
# dashboards
### Line Graph
Here's how to create a line chart in a Splunk dashboard from your data:

**1. Data Preparation with SPL:**

- **Ensure Proper Timestamp:** Splunk needs a proper timestamp (`_time`) for time-based charts. If your data doesn't have it, you'll need to use a function like `strptime` to create it. Assuming `minutly` represents minutes, here's an example:
    
    ```
    ... | eval _time=strptime(minutly, "%M") |  
    ```
    
- **Group by Speaker:** To get separate lines for each speaker, use `stats` to aggregate the `duration` by `speaker` over time.
    
    ```
    ... | stats sum(duration) as total_duration by speaker, _time | 
    ```
    
- **Complete SPL Query**
    
    ```
    your_base_search  | eval _time=strptime(minutly, "%M") | stats sum(duration) as total_duration by speaker, _time
    ```
    

**2. Create the Dashboard and Chart:**

- **Create a New Dashboard:**
    
    - In Splunk Web, go to "Search & Reporting".
        
    - Click "Dashboards" and then "Create New Dashboard".
        
    - Give your dashboard a title and choose a layout.
        
- **Add a Chart:**
    
    - In the dashboard editor, click "Add Panel".
        
    - Choose "Line Chart".
        
- **Configure the Search:**
    
    - In the chart configuration, enter the SPL query from Step 1 into the search box.
        
- **Set Chart Properties:**
    
    - **X-Axis:** Set to `_time`.
        
    - **Y-Axis:** Set to `total_duration`.
        
    - **Series:** set to `speaker`
        
- **Save:** Save the chart and the dashboard.
