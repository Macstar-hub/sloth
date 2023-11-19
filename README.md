# Table of Contents
1. Introduction
2. Template
   - Metadata
   - Spec Section
   - SLO Section
   - Formats
     - Perfomance Fortmat
       - Exampel
     - Avability Format
      - Example
   - Alerting Section
3. Examples
4. Compiling 
   - Manifest generate example
   - Manifest validate


# Introduction
* Each *SLA* or *SLO* is a CRD of K8s that will be transtlated via an operator for a Promtheus.

# Template
**Note:**
> :warning: the following has been designated as a template, and additional details will be determined in future sessions.
* The CRD have four sections as follows:

## Metadata
* In this section, we define specific labels required for some CRDs. For example, the Prometheus Operator is configured as shown below:
```yaml
name: xxxx-service-slo
annotations:
  meta.helm.sh/release-name: prometheus
  meta.helm.sh/release-namespace: default
creationTimestamp: "2023-10-31T17:04:34Z"
generation: 1
labels:
  prometheus: prometheus
  role: alert-rules
  app: sloth
  app: xxxx-service-slo
  app.kubernetes.io/instance: prometheus
  app.kubernetes.io/managed-by: Helm
  app.kubernetes.io/part-of: kube-prometheus-stack
  app.kubernetes.io/version: 52.1.0
  chart: kube-prometheus-stack-52.1.0
  heritage: Helm
  release: prometheus
namespace: default
```

## Spec Sections
* This section defines SLO parameters such as fixed and dynamic custom labels for improved SLO alerting, as shown in the labels below:
```yaml
labels:
  cluster: "production"
  component: "xxxx"
  context: "team-x"
```

## SLO Sections
* This section defines the SLO's actual defination.

* `objective:` As a percentage: 99.9 as percent. 
* `errorQuery:` The error query in Sloth is a Prometheus query that is used to calculate the number of events that occurred within a given time window that meet a specific error condition.
* `totalQuery:` In the context of Sloth, the "total query" refers to the Prometheus query that is used to calculate the total number of events for a given service and time window.
* `Formula:` SLO is calculated as a percentage using: SLO = (1 - (errorQuery / totalQuery)) * 100.
* `{{window}}:` In Sloth, a window refers to a specific time period over which Prometheus metrics are aggregated and evaluated for the purpose of calculating SLOs.
>**Note**
>> The expression {{.window}} is a variable in the Jinja2 template, and its value is set by the query configured in Grafana. There is no need for modification from the developer's side.

```yaml
- name: "xxxx-backend-slo"
  objective: 99.9
  description: "SLO based on availability for HTTP request responses."
  sli:
    events:
      errorQuery: sum(rate(http_request_duration_seconds_count{job="myservice",code=~"(5..|401)"}[{{.window}}]))
      totalQuery: sum(rate(http_request_duration_seconds_count{job="myservice"}[{{.window}}]))
  alerting:
    name: xxxxServiceHighErrorRate
    pageAlert:
      labels:
        severity: team-xxx ### or any custome value
    ticketAlert:
      labels:
        severity: warning
```

### Formats:
* We have categorized our Service Level Objectives (SLOs) into two main groups: Avability and Performance, each described as follows:
### Avability Format:
* The Uptime format, for example in REST services, is defined as the service's availability based on the ratio of the total count of Status Code 5xx to Status Code 2xx in specific time window.
* For example: 
```bash
sum(rate(http_request_duration_seconds_count{job="myservice",code=~"(5..|401)"}[{{.window}}]))/sum(rate(http_request_duration_seconds_count{job="myservice"}[{{.window}}]))
```
### Performance Format:
* The Performance format is defined by the number of requests with an average response time less than a specified threshold, for example, less than one second, relative to the total number of requests for the same service.
* For example: 
```bash
sum(rate(xxxx_duration_bucket{le="7.5"}[{{.window}}])) by (le) / ignoring(le) sum(rate(xxxx_duration_count[{{.window}}]))
```
## Alerting Section:
* This section defines custom alerting with fixed and dynamic labels, such as severity, and more.

```yaml
alerting:
  name: xxxxServiceHighErrorRate
  pageAlert:
    labels:
      severity: team-xxxx
  ticketAlert:
    labels:
      severity: warning
```
## Compiling (Command Usecase)
### How to generate prometheus rule file from sloth manifest:
```bash 
sloth generate -i exampleStatusCodeSLO.yaml -o /tmp/exampleStatusCodeSLOGenerated.yaml
copy /tmp/exampleStatusCodeSLOGenerated.yaml to destition prometheus rules.d
```
### How to validate sloth manifest:
```bash
sloth validate --input exampleStatusCodeSLO.yaml
```
**Note:**
> At the end, if there is any ambiguity, feel free to contact me :)
> MacStarBoy
