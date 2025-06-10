<!--- USED FOR WEBINAR 2025-06-11 --->

The 'main.py' file in the "average-panel-values" subfolder contains some boilerplate code that I would like to turn into a fully functioning quix app that consumes enriched data from solar panels and aggregates the solar panel data per location over 1-minute tumbling windows. This service will consume data that has already been enriched with weather configuration.

**Initial Task: Data Discovery and Schema Understanding**

1. **Connect and Consume:**  
   * You have access to a Kafka topic called `enriched_data` within the Quix environment, which you should specify by the environment variable `input`.  
   *  This topic contains JSON messages with solar panel telemetry already enriched with weather configuration data.  
   * Consume a representative sample of messages (e.g., up to 100 messages) from this topic   
     * \- use quix app stop conditions (count and timeout)  

2. **Analyze and Infer Schema:**  
   * Based on the consumed messages, analyze the structure and identify key fields relevant for aggregation. Pay particular attention to:  
     * A timestamp field (likely in nanoseconds or a parseable string).  
     * A field identifying the location (e.g., `location_id`).  
     * A field representing the power output of individual panels (e.g., `power_output`).  
     * A field identifying individual panels (e.g., `panel_id`), which will be needed to count unique panels per location.

**Service Logic (to be implemented *after* schema understanding):**

Based on your understanding of the data from the input topic:

1. **Identify Key Fields for Aggregation:**  
   * **Timestamp Field:** The field to be used for time-windowing.  
   * **Grouping Key:** The field to group by (this will be the location identifier, e.g., `location_id`).  
   * **Value to Aggregate:** The field representing the power output of a solar panel (e.g., `power_output`).  
   * **Panel Identifier:** The field identifying a unique solar panel (e.g., `panel_id`).  
2. **Time-Windowed Aggregation:**  
   * Implement a **1-minute tumbling window** aggregation.  
   * For each `location_id` within each 1-minute window, calculate:  
     * **`total_power_output`:** The sum of `power_output` from all panel messages for that location within the window.  
     * **`panel_count`:** The count of *unique* `panel_id` values seen for that location within the window.  
     * **`avg_power_per_panel`:** Calculated as `total_power_output / panel_count`.  
3. **State Management:**  
   * The service will need to maintain state for each active window and location to accumulate sums and unique panel counts.  
   * Ensure that state is managed correctly, especially for handling late-arriving data (if Quix's windowing functions handle this, use them; otherwise, consider a simple approach for this prompt). For this prompt, we can assume data arrives mostly in order for simplicity, but mention if a more robust solution for late data is trivial to implement.

**Output:**

1. **Output Topic:** The aggregated messages should be published to a Kafka topic specified by the environment variable called “output”, and set the default value (i.e set the topic name) to  “`downsampled_data”`  
2. **Output Message Content (per 1-minute window, per location):**  
     
   * `timestamp` (The end timestamp of the 1-minute window).  
   * `location_id` (The location for which the aggregation is performed).  
   * `location_name` (If available in the input, pass it through. If not, `location_id` is sufficient).  
   * `avg_power_per_panel` (float, the calculated average power per panel for that location in that window).  
   * `panel_count` (integer, the number of unique panels contributing to the average for that location in that window).  
   * `window_start` (The start timestamp of the 1-minute window \- optional but useful).  
   * `window_end` (The end timestamp of the 1-minute window \- same as `timestamp` above).

   
   **Example Output Structure:**

```json
{
    "timestamp": "2023-08-05T17:01:00.000000Z", // End of the window
    "location_id": "LONDON",
    "location_name": "London Farm", // if available
    "avg_power_per_panel": 145.75,
    "panel_count": 100,
    "window_start": "2023-08-05T17:00:00.000000Z",
    "window_end": "2023-08-05T17:01:00.000000Z"
}
```

**Environment:**

* The service will run within the Quix platform.  
* Use standard Quix Python library features for consuming from and producing to Kafka topics, and especially for its stateful processing and windowing capabilities.  
* Input and output topic names will be provided via environment variables \- when testing locally, create a “.env” file to store them.

Please generate the Python code for this Quix service and test it locally. Clearly state any assumptions made about input field names if they cannot be definitively inferred from a typical `enriched_data` stream.

To help you get started ask RunLLM for a complete end-to-end example of how to downsample some data.