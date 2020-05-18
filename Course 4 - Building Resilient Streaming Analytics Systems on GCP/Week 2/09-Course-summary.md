# Building Resilient Streaming Analytics Systems on GCP Course summary

## Course summary

### Week 1

- Streaming is data processing for unbounded datasets
  - What is streaming data, and the challenges associated with processing streaming data
  - It is important to be able to ingest various volumes of data because you may have spikes in data ingestion (in a commercial setting)
  - It is important to deal with latency because latency is a common real world problem
  - We are able to derive real-time insights from the data even as the data is streaming in
- Cloud Pub/Sub is a messaging service used as a starting point to process streaming data
  - [To derive real-time insights], we looked at the architecture that consisted of ingesting the data with Pub/Sub
- Cloud Dataflow provides a serverless service for processing batch and streaming data

### Week 2

- BigQuery allows you to stream records into a table
  - Streaming into BigQuery for durable storage and interactive analysis
- Cloud BigTable can handle NoSQL queries over large datasets (petabytes) in milliseconds
  - Cloud BigTable can be a much better solution when a much higher throughput is required
- Use BigQuery's analytic window functions for more advanced analytics
- BigQuery has built in GIS functionality
- Use BigQuery's `WITH` clause to help modularise your queries
- There are many ways to optimise your query performance