# Monitoring 

## How do we know when a system is healthy? 

A good Service Level Agreement (SLA) will define how the availability of a service is
measured using two key components: Service Level Objectives (SLO) and Service Level
Indicators. These components document what operations within a service are important, how
those operations are measured, and use those factors to define success targets that
establish a threshold for "healthiness". 

### Example Table of Service Level Objectives and Indicators 

**Component: Generic User Service**
<table>
<tbody><tr>
<td>
<p>Measurements</p>
</td>
<td>
<p>Metrics (SLIs)</p>
</td>
<td>
<p>SLOs</p>
</td>
</tr>
<tr>
<td>
<p>Login Success</p>
</td>
<td>
<p><code>Error Rate<br>
count of valid http
requests which do not result in a 200 status code / total http requests</code></p>

<p><code>Login Attempts<br>
count of total login
submissions</code></p>

<p><code>Total Successful Logins<span>count of
successful logins&nbsp;</span></code></p>
<p><code><span>Successful Login Rate<br>
Total Successful
logins / total login attempts</span></code></p>
</td>
<td>
<p>Error Rate: 30%&lt;<br>
<br>
Successful Login Rate: &gt;75%</p>
</td>
</tr>
<tr>
<td>
<p>Signup Success</p>
</td>
<td>
<p><code>Error Rate<br>
count of valid http
requests which do not result in a 200 status code / total http requests</code></p>

<p><code>Registration Submissions<br>
count of registration
submissions</code></p>

<p><code>Total Successful Registrations<span>count of
successful registration submissions&nbsp;</span></code></p>
<p><code><span>Successful Registration Submission Rate<br>
Registration
Submissions / Total Successful Submission Rate</span></code></p><code>
<p><code>Confirmation Rate<br>
Total Signup
Confirmations / Total Registration Submissions</code></p>
</code></td>
<td>
<p>Error Rate: 5%&lt;</p>

<p>Successful Registration Submission Rate: &gt;70%<br>
<span>Confirmation Rate: &gt;90%</span></p>
</td>
</tr>
<tr>
<td>
<p>Availability</p>
</td>
<td>
<p><code>Total Service Error Rate<br>
count of all valid
http requests which do not result in a 200 status code / total http
requests&nbsp;</code></p>
<p><code><span>Successful Transaction Rate<br>
avg(Confirmation Rate, Successful
Registration Submission Rate, Successful Login Rate)<br>
&nbsp;</span></code></p>
<p><code>Uptime<br>
Successful Health
Checks / Total Health Checks Initiated</code></p>

<p><code>Average Response Time<br>
Sum of response time
for all requests / request throughput</code></p>
</td>
<td>
<p>Total Service Error Rate: 10%&lt;<br>
&nbsp;</p>
<p><code>Successful Transaction Rate: &gt;80%<br>
<span>Average Response
Time: 500ms&lt;</span></code></p>
</td>
</tr>
</tbody></table>

Based on the table above we should be able to create alerting that is focused on critical
flows and metrics that represent the healthiness of your application. Performing the
analysis and documentation required to produce a good Service Level Agreement is time
consuming and is often an iterative process. In fact, most SLAs include a review interval
agreement that involves stakeholders coming together to improve or add onto the document,
because as services continue to evolve and change, so should your monitoring and
alerting. 

## Universal Standards 

Before Service Level Agreements can be fully fleshed out, its necessary to have some
monitoring and alerting in place if your application is already running production.
Luckily, there are industry standards we can leverage as a starting point, and when your
platform stack uses similar technologies, it's easy to build upon those standards to
implement pertinent coverage in your monitoring solutions 

### The Four Golden Signals 

As detailed in [Monitoring Distributed
Systems](https://sre.google/sre-book/monitoring-distributed-systems/#xref_monitoring_golden-signals)
in Google's Site Reliability Engineering book:  

Here is a succinct breakdown of each signal: 

* Latency: The time it takes to service a request.
* Traffic: A measure of how much demand is being placed on your system, usually
represented by transactions per second for your given system (http requests, network I/O,
events created, etc)
* Errors: The rate at which transactions fail.
* Saturation: Resource utilization (cpu, memory, io, etc) 

## Alerting 

Good observability of service health means nothing when you don't have good alerts to
match. Getting the proper stakeholders or remediation systems notified with the right
information as soon as possible is critical to maximizing the availability of your
system. Furthermore, it's important to respect the sanity of your engineers (because you
can't reboot an engineer), so avoiding common pitfalls such as avoiding [alert
fatigue](https://www.atlassian.com/incident-management/on-call/alert-fatigue) will also
be instrumental in the configuration of your alerts. 

### Alert Criticality 

Certain alerts will be more important than other alerts. A lot of organizations will have
4 or 5 levels of severity for their alerts. Personally, I think it's simpler to think
about these alerts as being either critical or a warning. Here is how I define the
distinction: 

* Critical alerts are actively causing customer impact, wake someone the hell up
* Warning alerts are indicative of potential customer impact, and your engineers can
likely wait for business hours to be addressed 

In a perfect world, our systems will always be in an either/or state as defined above.
Unfortunately, warnings can and will turn into critical issues, so it's important to
include escalation logic in your alerts to provide coverage if/when that transition
occurs. 

### Types of Monitors 

While different APMs might have different nomenclature for the types of monitors they
provide, I will provide my own generalized list here: 

* Static Threshold Monitors: This monitor will alert whenever a threshold is crossed.
* Example: Fire a critical alert when the hourly average CPU usage has been over a
threshold of 90% for the past 10 minutes.
* Algorithmic Monitors: These monitors will perform some calculation on a trend of your
metrics. These should usually be paired with a static threshold monitor.
* Change Monitors: How do my the last 1h of my metrics compare to the 1h average X days
ago?
* Anomaly Detection Monitors: These monitors use seasonal algorithms to calculate a range
based on historical data to determine an expected upper and lower bounds for your
metrics. 
* Forecast Monitors: These monitors are well suited for metrics with stable historical
trends. Gauging the rate of your historical metric trends, these monitors can forecast
the value of your metrics in the future, allowing you to alert before an issue might
arise.
* Predictive AI Monitors: These are usually proprietary monitors that use historical data
to detect atypical trends in your metrics.
* Examples: [Datadog's Watchdog](https://www.datadoghq.com/blog/watchdog/), [AppDynamics
Anomaly
Detection](https://docs.appdynamics.com/22.4/en/appdynamics-essentials/alert-and-respond/anomaly-detection/enable-and-configure-anomaly-detection),
[David by Dynatrace](https://www.dynatrace.com/platform/artificial-intelligence/), [Azure
Smart
Detection](https://docs.microsoft.com/en-us/azure/azure-monitor/app/proactive-diagnostics)
* Synthetic Monitors: These monitors provide end-to-end coverage by emulating a user's
journey across your application; any errors or performance bottlenecks that occur will
bubble up as alerts.
* Healthcheck Monitors: These monitors fire a series of requests at an interval to a
healthcheck endpoint, with configured expected status codes and/or response bodies 
* Real User Monitors: Using code included on the user's client, these monitors will
detect errors that user's experience in real-time
* Composite Monitors: These monitors combine the state of two or more other monitors to
gate whether or not an alert should be fired 

### Setting Thresholds 

Your approach to setting the proper thresholds for your alerts will change depending on
the nature of your metrics and the type of alert you're setting up. For instance, with
saturation metrics, there is usually a known upper bound based on total available
capacity. With these metrics, you'll want to set a critical threshold of 85-95% as they
approach their maximum usage. Before you fire a critical alert though, it's good setup a
warning threshold of impending maximum saturation to give your team the opportunity to
respond proactively vs. reactively.   
With metrics like total load, the thresholds can be less obvious. A higher then normal
spike in traffic usually shouldn't be your sole indicator of an issue in your system; in
this case, it's a good idea to create a composite alert: an alert with static thresholds,
and another alert (an algorithmic one) that fires when your metric crosses upper bounds
dynamically calculated by standard deviations based on historical data.   

### Actionable Alerts and Having STANDARDS 

Nothings worse than waking up at 3AM to an alert page that simply says "System A is in a
degraded state". Panic, delayed resolution, and burnout will ensue. It's important that
you treat yourself, especially when you're under pressure. Here is some simple standards
that should be included in your alerts: 

* Why am I getting this alert? A brief description of issue observed and what triggered
it.
* Where exactly is this issue occurring? Region, subscription, resource group, cluster
name, workload name, etc are all good examples of data points that should tell the
responder exactly where the problem is occurring.
* Remediation Steps. Some issues that arise have simple fixes, like "rollout restart the
deployment", or "Turn on and configure logrotate". Not all alerts will have a clear-cut
path to recovery, but at least provide a list of triage steps to help the responder
better diagnose and mitigate the issue.
* Playbook/Runbook Links. These are very nice to have if available. A good runbook will
include ongoing and past problems, steps for recovery, links to their location, IT
owners, and architecture diagrams.
* How do I know I fixed it? Provide detailed steps to perform validation confirming that
the issue has been resolved.
