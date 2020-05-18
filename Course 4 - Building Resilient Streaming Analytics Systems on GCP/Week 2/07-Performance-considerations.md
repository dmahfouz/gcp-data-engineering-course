# Performance Considerations

## Agenda

- GIS Functions
- `WITH` clauses vs. Permanent tables
- Analytical window functions
- Ranking functions + `ARRAY`s
- **Performance considerations**

## Improve scalability by improving efficiency

There are five key areas for performance optimisation in BigQuery

- **I/O (input/output)** - *how many bytes were read from disk?*
- **Shuffling** - *how many bytes were passed to the next query stage?*
- **Grouping** - *how many bytes were passed through each group?*
- **Materialisation** - *how many bytes were written permanently out to disk?*
- **Functions and UDFs** - *how computationally expensive is the query on your CPU?*

![imprv_efficiency](imgs/perf-considerations/improve-efficiency.jfif)

## Optimising BigQuery queries cheat-sheet

1. Don't select more data columns than you need (avoid `SELECT * ..` at all costs when you can)
2. If you have a very large dataset, consider approximate (`APPROX_[..]`) functions instead of regular ones
3. Make liberal use of the `WHERE` clause at all times to filter data
4. **Don't** use an `ORDER BY` on a `WITH` clause or any sub-queries, only use `ORDER BY` as the last thing you do
5. For `JOIN`s, put the **larger table on the left**, if you can - this will help BigQuery perform optimisations for you
6. You can use wildcards in table suffixes to query multiple tables, **but**, try to be as specific as possible with those wildcards
7. Avoid using `GROUP BY`s over a large number of distinct values or columns with high cardinality (e.g. Wikipedia authors) - this is bad practice / an anti-pattern, instead use low unique value `GROUP BY`s
8. Use table partitioning as often as you can

![cheat-sheet](imgs/perf-considerations/cheat-sheet.jfif)

## BigQuery supports partitioning tables

There are two main ways to partition tables in BigQuery

1. When you first ingest the data as a destination table or as an actual column that you can partition on

2. Or, as an actual column you can partition on

### Examples

- The first example shows that you create a table at ingestion time paritioning by day
- Examples 2 and 3 show how to promote an existing column you have in your dataset to a partition column
    - The columns must be of type dates, timestamp or newly added as an integer range column

![partition](imgs/perf-considerations/partition.jfif)

### Partitioning can improve query cost and performance by reducing the data that's being scanned

- Since BigQuery has access to the partitioning metadata, it can avoid partitions of data that it knows will be of zero value to the query asked, like ignoring all data before a certain date filter, as you can see here

![partition2](imgs/perf-considerations/partition2.jfif)