---
title: SigNoz - Open-source alternative to DataDog
slug: open-source-datadog-alternative
date: 2021-08-25
tags: [application-monitoring, datadog, apm-tools]
author: Pranay Prateek
author_title: SigNoz Team
author_url: https://github.com/pranay01
author_image_url: https://avatars.githubusercontent.com/u/504541?v=4
description: DataDog is a popular APM tool. But it is very expensive and opaque about its billing practices. What if you could get a SaaS like experience from an open-source APM tool....
image: /img/blog/2021/08/signoz_datadog_alternatives_cover-min.png
keywords:
  - datadog
  - open source
  - datadog alternative
  - application monitoring
  - apm tools
  - signoz
---

More and more companies are now shifting to a cloud-native & microservices-based architecture. Having an application monitoring tool is critical in this world because you can’t just log into a machine and figure out what’s going wrong.

<!--truncate-->

![cover image](/img/blog/2021/08/signoz_datadog_alternatives_cover-min.png)

We have spent years learning about application monitoring & observability. What are the key features an observability tool should have to enable fast resolution of issues.

In our opinion, good observability tools should have

- Out of the box application metrics
- Way to go from metrics to traces to find why some issues are happening
- Seamless flow between metrics, traces & logs — the three pillars of observability
- Filtering of traces based on different tags and filters
- Ability to set dynamic thresholds for alerts
- Transparency in pricing

## User experience not great in current open-source tools

We found that though there are open-source tools like Prometheus & Jaeger, they don’t provide a great user experience as SaaS products do. It takes lots of time and effort to get them working, figuring out the long-term storage, etc. And if you want metrics and traces, it’s not possible as Prometheus metrics & Jaeger traces have different formats.

SaaS tools like DataDog and NewRelic do a much better job at many of these aspects:

- They are easy to setup & get started
- Provide out-of-box application metrics
- Provides correlation between metrics & traces

But it has the following issues:

- Crazy node-based pricing, which doesn’t make sense in today’s micro-services architecture. Any node which is live for more than 8hrs in a month is charged. So, unsuitable for spiky workloads
- Very costly. They charge custom metrics for $5/100 metrics
- It is cloud-only, so not suitable for companies that have concerns with sending data outside their infra
- For any small feature, you are dependent on their roadmap. We think this is an unnecessary restriction for a product which developers use. A product used by developers should be extendible

---

To fill this gap we built [SigNoz](https://signoz.io/), an open-source alternative to DataDog.

## Key Features of SigNoz - a DataDog alternative

Some of our key features which makes SigNoz vastly superior to current open-source products and a great alternative to DataDog are:

- Out of the box application metrics
- Seamless flow between metrics & traces
- Filtering based on tags
- Custom aggregates on filtered traces
- Transparent usage Data
- Detailed Flamegraphs & Gantt charts

### Out of the box application metrics

Get p90, p99 latencies, RPS, Error rates and top endpoints for a service out of the box.

import Screenshot from "@theme/Screenshot"

<Screenshot
    alt="SigNoz dashboard showing popular RED metrics"
    height={500}
    src="/img/blog/common/signoz_charts_application_metrics.png"
    title="SigNoz UI showing application overview metrics like RPS, 50th/90th/99th Percentile latencies, and Error Rate"
    width={700}
/>

### Seamless flow between metrics & traces

Found something suspicious in a metric, just click that point in the graph & get details of traces which may be causing the issues. Seamless, Intuitive.

<Screenshot
    alt="Seamless flow between metrics and traces"
    height={500}
    src="/img/blog/2021/08/metrics_to_traces_signoz-min.png"
    title="Move from metrics to traces at any point of time which needs more analysis"
    width={700}
/>

### Filtering based on tags

For example, you can find latency experienced by customers who have customer_type set as `premium`.

<Screenshot
    alt="Filtering based on tags"
    height={500}
    src="/img/blog/2021/08/tags_based_filtering_signoz-min.png"
    title="Filter traces for a specific user group using tags"
    width={700}
/>

### Custom aggregates on filtered traces

Create custom metrics from filtered traces to find metrics of any type of requests. Want to find p99 latency of `customer_type: premium` who are seeing `status_code:400`. Just set the filters, and you have the graph. Boom!

<Screenshot
    alt="Custom aggregates on filtered traces"
    height={500}
    src="/img/blog/2021/08/metrics_on_filtered_traces-min.png"
    title="Find custom aggregates on filtered traces"
    width={700}
/>

### Transparent usage Data

You can drill down details of how many events is each application sending or at what granularity, so that you can adjust your sampling rate as needed and not get a shock at the end of the month ( case with SaaS vendors many a times)

<Screenshot
    alt="Transparent usage data"
    height={500}
    src="/img/blog/2021/08/transparent_usage_data-min.png"
    title="SigNoz provides usage explorer so that you are always informed about your usage"
    width={700}
/>

### Detailed Flamegraphs & Gantt charts

Detailed flamegraph & Gantt charts to find exact cause of the issue, and which of the underlying requests is causing the problem. Is it a SQL query gone rogue or a redis operation is causing an issue?

<Screenshot
    alt="Detailed Flamegraphs & Gantt charts"
    height={500}
    src="/img/blog/common/signoz_flamegraphs.png"
    title="Spans of a trace visualized with the help of flamegraphs and gantt charts in SigNoz dashboard"
    width={700}
/>

---

## Getting started with SigNoz

If you have docker installed, getting started with SigNoz just takes three easy steps at the command line:

```
git clone https://github.com/SigNoz/signoz.git
cd signoz/deploy/
./install.sh
```

You can read more about deploying SigNoz from its [documentation](https://signoz.io/docs/deployment/docker/).

If you liked what you read, then check out our GitHub repo 👇

[![SigNoz GitHub repo](/img/blog/common/signoz_github.png)](https://github.com/SigNoz/signoz)

You can get in touch with us, and we will be happy to help. Write to us at: support@signoz.io

Our slack community is a great place to get your queries solved instantly and get community support for SigNoz. Link to join 👇<br></br>
[SigNoz slack community](https://bit.ly/signoz-slack)

If you want to read more about SigNoz 👇<br></br>
[Monitor Spring Boot application with OpenTelemetry and SigNoz](https://signoz.io/blog/opentelemetry-spring-boot/)