---
id: observability
title: Python SDK developer's guide - Observability
sidebar_label: Observability
sidebar_position: 3
description: The Observability section of the Temporal Developer's guide covers the many ways to view the current state of your Temporal Application—that is, ways to view what Workflow Executions are tracked by the Platform and the state of any given Workflow Execution, either currently or at points of an execution.
toc_max_heading_level: 4
tags:
- guide-context
- developer-guide
- sdk
- python
- how-to
---

<!-- THIS FILE IS GENERATED. DO NOT EDIT THIS FILE DIRECTLY -->

The observability section of the Temporal Developer's guide covers the many ways to view the current state of your <a class="tdlp" href="/temporal#temporal-application">Temporal Application<span class="tdlpiw"><img src="/img/link-preview-icon.svg" alt="Link preview icon" /></span><span class="tdlpc"><span class="tdlppt">What is a Temporal Application</span><br /><br /><span class="tdlppd">A Temporal Application is a set of Workflow Executions.</span><span class="tdlplm"><br /><br /><a class="tdlplma" href="/temporal#temporal-application">Learn more</a></span></span></a>—that is, ways to view which [Workflow Executions](/workflows#workflow-execution) are tracked by the <a class="tdlp" href="/temporal#temporal-platform">Temporal Platform<span class="tdlpiw"><img src="/img/link-preview-icon.svg" alt="Link preview icon" /></span><span class="tdlpc"><span class="tdlppt">What is the Temporal Platform?</span><br /><br /><span class="tdlppd">The Temporal Platform consists of a Temporal Cluster and Worker Processes.</span><span class="tdlplm"><br /><br /><a class="tdlplma" href="/temporal#temporal-platform">Learn more</a></span></span></a> and the state of any specified Workflow Execution, either currently or at points of an execution.

:::info WORK IN PROGRESS

This guide is a work in progress.
Some sections may be incomplete or missing for some languages.
Information may change at any time.

If you can't find what you are looking for in the Developer's guide, it could be in [older docs for SDKs](https://legacy-documentation-sdks.temporal.io/).

:::

This section covers features related to viewing the state of the application, including:

- [Metrics](#metrics)
- [Tracing](#tracing)
- [Logging](#logging)
- [Visibility](#visibility)

## Metrics

Each Temporal SDK is capable of emitting an optional set of metrics from either the Client or the Worker process.
For a complete list of metrics capable of being emitted, see the <a class="tdlp" href="/references/sdk-metrics#">SDK metrics reference<span class="tdlpiw"><img src="/img/link-preview-icon.svg" alt="Link preview icon" /></span><span class="tdlpc"><span class="tdlppt">SDK metrics</span><br /><br /><span class="tdlppd">The Temporal SDKs emit metrics from Temporal Client usage and Worker Processes.</span><span class="tdlplm"><br /><br /><a class="tdlplma" href="/references/sdk-metrics#">Learn more</a></span></span></a>.

Metrics can be scraped and stored in time series databases, such as:

- [Prometheus](https://prometheus.io/docs/introduction/overview/)
- [M3db](https://m3db.io/docs/)
- [statsd](https://github.com/statsd/statsd)

Temporal also provides a dashboard you can integrate with graphing services like [Grafana](https://grafana.com/docs/). For more information, see:

- Temporal's implementation of the [Grafana dashboard](https://github.com/temporalio/dashboards)
- [How to export metrics in Grafana](https://github.com/temporalio/helm-charts#exploring-metrics-via-grafana)

Metrics in Python are configured globally; therefore, you should set a Prometheus endpoint before any other Temporal code.

The following example exposes a Prometheus endpoint on port `9000`.

```python
from temporalio.runtime import Runtime, TelemetryConfig, PrometheusConfig

# Create a new runtime that has telemetry enabled. Create this first to avoid
# the default Runtime from being lazily created.
new_runtime = Runtime(telemetry=TelemetryConfig(metrics=PrometheusConfig(bind_address="0.0.0.0:9000")))
my_client = await Client.connect("my.temporal.host:7233", runtime=new_runtime)
```

## Tracing

Tracing allows you to view the call graph of a Workflow along with its Activities and any Child Workflows.

Temporal Web's tracing capabilities mainly track Activity Execution within a Temporal context. If you need custom tracing specific for your use case, you should make use of context propagation to add tracing logic accordingly.

For information about Workflow tracing, see [Tracing Temporal Workflows with DataDog](https://spiralscout.com/blog/tracing-temporal-workflow-with-datadog).

For information about how to configure exporters and instrument your code, see [Tracing Temporal Services with OTEL](https://github.com/temporalio/temporal/blob/master/develop/docs/tracing.md).

To configure tracing in Python, install the `opentelemetry` dependencies.

```bash
# This command installs the `opentelemetry` dependencies.
pip install temporalio[opentelemetry]
```

Then the [`temporalio.contrib.opentelemetry.TracingInterceptor`](https://python.temporal.io/temporalio.contrib.opentelemetry.TracingInterceptor.html) class can be set as an interceptor as an argument of [`Client.connect()`](https://python.temporal.io/temporalio.client.Client.html#connect).

When your Client is connected, spans are created for all Client calls, Activities, and Workflow invocations on the Worker.
Spans are created and serialized through the server to give one trace for a Workflow Execution.

## Logging

Send logs and errors to a logging service, so that when things go wrong, you can see what happened.

The SDK core uses `WARN` for its default logging level.

You can log from a Workflow using Python's standard library, by importing the logging module `import logging`.

Set your logging configuration to a level you want to expose logs to.
The following example sets the logging information level to `INFO`.

```python
logging.basicConfig(level=logging.INFO)
```

Then in your Workflow, set your [`logger`](https://python.temporal.io/temporalio.workflow.html#logger) and level on the Workflow. The following example logs the Workflow.

```python
@workflow.defn
class SayHelloWorkflow:
    @workflow.run
    async def run(self, name: str) -> str:
        workflow.logger.info(f"Running workflow with parameter {name}")
        return await workflow.execute_activity(
            your_activity, name, start_to_close_timeout=timedelta(seconds=10)
        )
```

The following is an example output:

```
INFO:temporalio.workflow:Running workflow with parameter Temporal ({'attempt': 1, 'your-custom-namespace': 'default', 'run_id': 'your-run-id', 'task_queue': 'your-task-queue', 'workflow_id': 'your-workflow-id', 'workflow_type': 'SayHelloWorkflow'})
```

:::note

Logs are skipped during replay by default.

:::

### Custom logger

Use a custom logger for logging.

Use the built-in [Logging facility for Python](https://docs.python.org/3/library/logging.html) to set a custom logger.

## Visibility

The term Visibility, within the Temporal Platform, refers to the subsystems and APIs that enable an operator to view Workflow Executions that currently exist within a Cluster.

### Search Attributes

The typical method of retrieving a Workflow Execution is by its Workflow Id.

However, sometimes you'll want to retrieve one or more Workflow Executions based on another property. For example, imagine you want to get all Workflow Executions of a certain type that have failed within a time range, so that you can start new ones with the same arguments.

You can do this with <a class="tdlp" href="/visibility#search-attribute">Search Attributes<span class="tdlpiw"><img src="/img/link-preview-icon.svg" alt="Link preview icon" /></span><span class="tdlpc"><span class="tdlppt">What is a Search Attribute?</span><br /><br /><span class="tdlppd">A Search Attribute is an indexed name used in List Filters to filter a list of Workflow Executions that have the Search Attribute in their metadata.</span><span class="tdlplm"><br /><br /><a class="tdlplma" href="/visibility#search-attribute">Learn more</a></span></span></a>.

- <a class="tdlp" href="/visibility#default-search-attributes">Default Search Attributes<span class="tdlpiw"><img src="/img/link-preview-icon.svg" alt="Link preview icon" /></span><span class="tdlpc"><span class="tdlppt">What is a Search Attribute?</span><br /><br /><span class="tdlppd">A Search Attribute is an indexed name used in List Filters to filter a list of Workflow Executions that have the Search Attribute in their metadata.</span><span class="tdlplm"><br /><br /><a class="tdlplma" href="/visibility#default-search-attributes">Learn more</a></span></span></a> like `WorkflowType`, `StartTime` and `ExecutionStatus` are automatically added to Workflow Executions.
- _Custom Search Attributes_ can contain their own domain-specific data (like `customerId` or `numItems`).
  - A few <a class="tdlp" href="/visibility#custom-search-attributes">generic Custom Search Attributes<span class="tdlpiw"><img src="/img/link-preview-icon.svg" alt="Link preview icon" /></span><span class="tdlpc"><span class="tdlppt">What is a Search Attribute?</span><br /><br /><span class="tdlppd">A Search Attribute is an indexed name used in List Filters to filter a list of Workflow Executions that have the Search Attribute in their metadata.</span><span class="tdlplm"><br /><br /><a class="tdlplma" href="/visibility#custom-search-attributes">Learn more</a></span></span></a> like `CustomKeywordField` and `CustomIntField` are created by default in Temporal's [Docker Compose](/kb/all-the-ways-to-run-a-cluster#docker-compose).

The steps to using custom Search Attributes are:

- Create a new Search Attribute in your Cluster in the CLI or Web UI.
  - For example: `temporal operator search-attribute create --name CustomKeywordField --type Text`
    - Replace `CustomKeywordField` with the name of your Search Attribute.
    - Replace `Text` with a type value associated with your Search Attribute: `Text` | `Keyword` | `Int` | `Double` | `Bool` | `Datetime` | `KeywordList`
- Set the value of the Search Attribute for a Workflow Execution:
  - On the Client by including it as an option when starting the Execution.
  - In the Workflow by calling `UpsertSearchAttributes`.
- Read the value of the Search Attribute:
  - On the Client by calling `DescribeWorkflow`.
  - In the Workflow by looking at `WorkflowInfo`.
- Query Workflow Executions by the Search Attribute using a <a class="tdlp" href="/visibility#list-filter">List Filter<span class="tdlpiw"><img src="/img/link-preview-icon.svg" alt="Link preview icon" /></span><span class="tdlpc"><span class="tdlppt">What is a List Filter?</span><br /><br /><span class="tdlppd">A List Filter is the SQL-like string that is provided as the parameter to an Advanced Visibility List API.</span><span class="tdlplm"><br /><br /><a class="tdlplma" href="/visibility#list-filter">Learn more</a></span></span></a>:
  - [In the Temporal CLI](/cli/operator#list-2)
  - In code by calling `ListWorkflowExecutions`.

Here is how to query Workflow Executions:

Use the [list_workflows()](https://python.temporal.io/temporalio.client.Client.html#list_workflows) method on the Client handle and pass a <a class="tdlp" href="/visibility#list-filter">List Filter<span class="tdlpiw"><img src="/img/link-preview-icon.svg" alt="Link preview icon" /></span><span class="tdlpc"><span class="tdlppt">What is a List Filter?</span><br /><br /><span class="tdlppd">A List Filter is the SQL-like string that is provided as the parameter to an Advanced Visibility List API.</span><span class="tdlplm"><br /><br /><a class="tdlplma" href="/visibility#list-filter">Learn more</a></span></span></a> as an argument to filter the listed Workflows.

```python
async for workflow in client.list_workflows('WorkflowType="MyWorkflowClass"'):
    print(f"Workflow: {workflow.id}")
```

### Custom Search Attributes

After you've created custom Search Attributes in your Cluster (using `tctl search-attribute create`or the Cloud UI), you can set the values of the custom Search Attributes when starting a Workflow.

To set custom Search Attributes, use the `search_attributes` parameter of the ['start_workflow()'](https://python.temporal.io/temporalio.client.Client.html#start_workflow) method.

```python
handle = await client.start_workflow(
    "your-workflow-name",
    id="your-workflow-id",
    task_queue="your-task-queue",
    search_attributes={"Your-Custom-Keyword-Field": ["value"]},
)
```

### Upsert Search Attributes

You can upsert Search Attributes to add or update Search Attributes from within Workflow code.

To upsert custom Search Attributes, use the [`upsert_search_attributes()`](https://python.temporal.io/temporalio.workflow.html#upsert_search_attributes) method.

The keys are added to or replace the existing Search Attributes, similar to [`dict.update()`](https://docs.python.org/3/library/stdtypes.html#dict.update).

```python
workflow.upsert_search_attributes({"Your-Custom-Keyword-Field": ["new-value"]})
```

### Remove Search Attribute

To remove a Search Attribute that was previously set, set it to an empty array: `[]`.

To remove a Search Attribute, use the [`upsert_search_attributes()`](https://python.temporal.io/temporalio.workflow.html#upsert_search_attributes) function with an empty list as its value.

```python
workflow.upsert_search_attributes({"Your-Custom-Keyword-Field": []})
```
