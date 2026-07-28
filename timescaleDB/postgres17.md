# TimescaleDB Deployment Instructions

This doc shows how to deploy postgres17 as OEDs Database docker container.

## Pull & Deploy Docker container

> Recommended to start from a fresh OED install to not corrupt/conflict `postgres-data/`

1. pull the specific docker image using the version tag ```docker pull postgres:17.5```
        SHL: I this necessary? I thought it would automatically happen with the next step.
2. From your fresh repo of OED edit `containers/database/Dockerfile` and update the docker tag definition
   `FROM postgres:15.3`->`FROM postgres:17.5`
3. Then follow the typical steps to start up OED.

## Testing Postgres 17.5 compared to 15.3 (current)

    SHL: Is this for others to do when they upgrade or not done yet for this document? If for others then how to do it would be nice (at some point).

1. Compare boot and init processing time for materialized views

```text
    Results:
```

2. Confirm that migrating existing OED data is possible
3. Confirm which materialized views and query functions work in 17
