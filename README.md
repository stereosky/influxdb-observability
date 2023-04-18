# InfluxDB Observability

This repository is a reference for converting observability signals (traces, metrics, logs) to/from a common InfluxDB schema.

## Demo

Steps to run the current demo follow.

In an InfluxDB Cloud 2 account backed by IOx, create a bucket named `otel` and a token with permission to read and write to that bucket.

In demo/docker-compose.yml, set values for these keys.
The key `INFLUXDB_BUCKET_ARCHIVE` is optional;
if set, it should point to an InfluxDB bucket with longer retention policy than `INFLUXDB_BUCKET`,
so that the "Archive Trace" button in Jaeger works properly:

The community addition focuses on the useability of the demo with Grafana. With this being said to improve demo setup we have introduced a `.env` file that will allow you to set the following variables:

```bash
export INFLUXDB_ADDR=eu-central-1-1.aws.cloud2.influxdata.com
export INFLUXDB_TOKEN=
export INFLUXDB_ORG=Jay-IOx
export INFLUXDB_BUCKET=otel 
export INFLUXDB_BUCKET_ARCHIVE=otel-archive
```

Make sure this file exists in the root of the project. Then also make sure you run the below commands in the root of the project aswell.


Build the needed docker images:
```console
$ docker compose --file demo/docker-compose.yml --project-directory . build
```

Run the docker compose:
```console
$ docker compose --file demo/docker-compose.yml --project-directory . up --abort-on-container-exit --remove-orphans
```

Traces are generated by "HotRod", an application designed to demonstrate tracing.
Browse to HotRod at http://localhost:8080 and click some buttons to trigger trace activity.

Query those traces.
Browse to Jaeger at http://localhost:16686 and click "Find Traces" near the bottom left.

Click any trace.

View the dependency graph.
Click "System Architecture".

Grafana is available at http://localhost:3000. The default username and password are both `admin`. The default datasource of flightSQL is already confgiured.

**Note: You can find a dashboard to import under `demo/grafana/dashboards/Open Telemetry-1681814438598.json`**

If you would like to access the Trace node tree. Then Make sure to enable it within the Jaeger datasource. Head to data sources and click on the Jaeger datasource. Then enable `Enable Node Graph`. Then click save and test.


The images `otelcol-influxdb` and `jaeger-influxdb` are automatically built and pushed to Docker at https://hub.docker.com/r/jacobmarble/otelcol-influxdb and https://hub.docker.com/r/jacobmarble/jaeger-influxdb .

## Schema Reference

[Schema reference with conversion tables](docs/index.md).

## Modules

### `common`

The golang package `common` contains simple utilities and common string values,
used in at least two of the above-mentioned packages.

### `otel2influx` and `influx2otel`

The golang package [`otel2influx`](otel2influx/README.md) converts OpenTelemetry protocol buffer objects to (measurement, tags, fields, timestamp) tuples.
It is imported by [the OpenTelemetry Collector InfluxDB exporter](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/exporter/influxdbexporter)
and by [the Telegraf OpenTelemetry input plugin](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/opentelemetry).

The golang package [`influx2otel`](influx2otel/README.md) converts (measurement, tags, fields, timestamp) tuples to OpenTelemetry protocol buffer objects.
It is imported by [the OpenTelemtry Collector InfluxDB receiver](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/receiver/influxdbreceiver)
and by [the Telegraf OpenTelemetry output plugin](https://github.com/influxdata/telegraf/tree/master/plugins/outputs/opentelemetry).

### `jaeger-influxdb`

The [Jaeger Query Plugin for InfluxDB](jaeger-influxdb) enables querying traces stored in InfluxDB/IOx via the Jaeger UI.

### `tests-integration`

The golang package `tests-integration` contains integration tests.
These tests exercise the above packages against OpenTelemetry Collector Contrib and Telegraf.

To run these tests:
```console
$ cd tests-integration
$ go test
```

## Contributing

Changes can be tested on a local branch using the `run-checks.sh` tool.
`run-checks.sh` verifies `go mod tidy` using `git diff`,
so any changes must be staged for commit in order for `run-checks.sh` to pass.

To update critical dependencies (OpenTelemetry, Jaeger, and intra-repo modules) in the various modules of this repository:
- run `update-deps.sh`
- stage the changed `go.mod` and `go.sum` files
- run `run-checks.sh`

## TODO
Fork this demo:
https://github.com/open-telemetry/opentelemetry-demo

