# TimescaleDB Deployment Instructions

This doc shows how to deploy the timescaleDB extension on top of Postgres17 as the OED Database docker container.
*The reason for deploying with Postgres 17 was to confirm the latest version of timescale was enabled. Tangentially refer to [postgres17.md](./postgres17.md) docs for notes on deployment and upgrade help.*

## Pull & Deploy Docker container

> Recommended to start from a fresh OED install to not corrupt/conflict `postgres-data/`. These directions are for a specific version but others can be used.

1. pull the specific docker image using the version tag ```docker pull timescale/timescaledb:2.21.1-pg17```

        SHL: I this necessary? I thought it would automatically happen with the next step.

2. From your fresh repo of OED edit `containers/database/Dockerfile` and update the docker tag definition
   `FROM postgres:15.3`->`FROM timescale/timescaledb:2.21.1-pg17`
3. Then follow the typical steps to start up OED.

### TimescaleDB findings

1. In order to take advantage of the timescaleDB OED need to create or adapt a table to use the hypertable function. Setting chunk size is how the table indexes time and still considering `tsdb.segmentby = 'meter'`. SHL: Can this be expanded on. Does it mean it is an alternative and to what? SHL END The documents may help explain about  hypertables: [hypertable Doc](https://docs.tigerdata.com/use-timescale/latest/hypertables/), [Create HyperTable Doc](https://docs.tigerdata.com/api/latest/hypertable/create_hypertable/#arguments) and [Optomization Docs](https://docs.tigerdata.com/use-timescale/latest/hypertables/improve-query-performance/#optimize-hypertable-chunk-intervals/).

    ``` sql
        SHL: Can OED avoid making a copy and do alter? If the current table was modified to use a TIMESTAMPTZ (maybe with a default of UTC) would that work? 
    -- Create a copy of the readings table that can meddle with and create a new column for hypertable to support (requires TIMESTAMPTZ).
    CREATE TABLE h_reading AS 
    TABLE readings;
    ALTER TABLE h_reading
    ADD COLUMN time TIMESTAMPTZ;
    
    UPDATE h_reading SET time = start_time AT TIME ZONE 'PDT';
        SHL: Why is this PDT?
    SELECT create_hypertable('h_reading', 'time')
    WITH (
       tsdb.hypertable,
       tsdb.partition_column='time',
       tsdb.chunk_interval='1 hour'
    );
     ```

2. The thing still being toying with are the settings for continuous_aggregate policy. start and end offset is the window of time this policy will look to. Schedule interval defaults to hourly

    ```sql
    SELECT add_continuous_aggregate_policy(
        'h_reading',
        start_offset => INTERVAL 'your_start_offset',
        end_offset => INTERVAL 'your_end_offset',
        schedule_interval => INTERVAL 'your_schedule_interval'
    );
    ```

generate_series
: can't be used post hypertable but hopefully can still be used for

tsrange
: still supported, but don't utilize hypertable indexing. The alternate is time_bucket.

continuous_aggregate
: The timescale cron that checks if new rows have entered its table and begins processing the missing rows.

SHL: How does this potentially relate to automatic updates and they possible? Is that what is next?

Next steps to test timescale

```txt
Steps for automating the view
1. add some more readings and making sure it refreshes quickly
2. getting timescale to work with Meter hourly view
    SHL: Can you elaborate on this?
3. quantity of views up (including raw)
4. indexes for faster readings access. 
5. Manual earliest and latest views (If they’re faster how much faster are these things).
    SHL: More information on this?

Goals
Meter hourly view (Readings come from there)
    SHL: Is this only when you want hourly readings?
Daily meter is made by aggregating hourly.
Group hourly is done by agg meter hourly.
35k readings per year (under a minute)
    SHL: where did this goal come from and what does it imply?
```
