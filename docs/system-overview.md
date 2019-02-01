# AO System Overview

The system is built to accept a webhook from Azure Monitor, and then consult a set of rules, optionally sending a notification to MS Teams for every matching rule/condition.

The flow looks like this:

- Receive message and turn into an `InNotification`
- See if any `NotificationRules` match
- For each match run through the processing logic and optionally send any `OutNotification`s.

The simplest example of an alert rule is:

```csharp
public bool IsMatch(InNotification notification) {
    return true;
}

public async Task EvaluateNotification(InNotification notification, ILogger log) {
    log.Trace($"The always true logger was called for {notification.Name}");
}
```

Each rule is designed to have a _fast_ evaluation method `IsMatch`, and a second - potentially slower - `EvaluateNotification` method.  They might be called like this (pseudocode):

```csharp
var inNotification = processWebhookFromAzureMonitor(payload);
var fastMatches = GetRules().Where(rule => rule.IsMatch(inNotification));
foreach(var match in fastMatches)
{
    await match.EvaluateNotification(inNotification);
}
```

At some point the system will support parallel evaluation (potentially via the durable task framework) so that evaluations can potentially be complex and expensive, querying external resources to determine if there should be a trigger.

---

## Old stuff

Kept for inspiration.

---

The system is built as a set of [Azure Functions].  The functions chain using [Durable Functions] rather than wiring up the intermedia storage queues.

The system reacts to *notifications* triggered by external systems.  These are ingested by calling an endpoint, and are most probably webhooks.  The flow of an notification is typically:

- Monitoring sink endpoint called
- Monitor payload processed and turned into a standard `InNotification` object
- `NotificationRules` are evaluated to determine if the monitor should generate an `OutNotification`
- Each `OutNotification` is processed, and zero or more outputs are generated

>An OutNotification might generate zero outputs if some kind of flow control/dedupe/mute is in place

## NotificationRule
Notifcation rules are comprised of (mandatory):

- An out notification target

Typical:
- A MonitorId criteria (each incoming monitor has a unique id)

Optional:
- Criteria to evaluate the notification (no criteria means always true)
- A filter for the dimensions of the notification (e.g. server name)
- A name and description

An incoming notification (e.g. `Server-CPU`) can have multiple notification rules attached to it.  This might look like:

```json
{
    "NotificationRules": [
        {   
            "NotificationDescription": "Server CPU - All Servers",
            "Conditions": [
                { "InNotififcationId": "Server-CPU" }
            ], 
            "OutNotifications": [{
                "TargetId": "Slack",
                "Channel": "#proudctionisburning"
            }]
        }
        ,{  
            "NotificationDescription": "Server CPU - Web Servers",
            "Conditions": [
                { "ServerName": "web" },
                { "InNotificationId": "Server-CPU" }
            ],
            "OutNotifications": [{
                "TargetId": "Slack",
                "Channel": "#webteam",
                "Message": "Web CPU is high"
            }]
        }
    ]
}
```

It is also possible to create an AlertRule with no criteria - this will be matched for every incoming monitor (and so could be used to ensure all alerts are routed to a specified log).

## Out Notifications

Supported notifications and their configuration details:

- Slack
  - Uri
  - ApiKey
  - Channel
  - Message
- MS Teams (channel connector)
  - Uri
  - Message
- Webhook
  - Uri
  - Message
- Log
  - Message

## Outstanding questions
- What does testing look like?  Both in terms of testing the config isn't broken (a bad deploy of a config renders the app useless), and in terms of any of the individual functions work.
  - Because notification rules are so flexible (maybe the notification doesn't even have a metric, it might go to an SMS, Webhook JSON payload, Slack, MS Teams channel, PagerDuty), testing definitions is important (both when rules are changed, and code that handles them is updated)
- What does testing in production look like?

## Answered questions
- What does local testing look like?
  - Durable function testing works fine with the storage emulator (verified with functions 1.0.24 and DurableTask 1.7.0).  Requires an explicit reference to Newtonsoft.Json 11.0.2.
- Notification configuration split
  - `OutNotificationTarget` contains configuration data - e.g. for MS Teams - Id, Endpoint Uri
  - `NotificationRule` contains details to be sent to the `NotificationTarget`, e.g. Message, CardType
  - Limited replacement tokens (e.g. METRIC_AVG, METRIC_MIN_MAX_AVG)
- If a new orchestrator function is deployed the hub name needs to be changed so that new executions will get dedicated replays

## InNotification Types
- Some notifications will be metric based - a value exceeding/matching/etc. a threshold
- Some notifications will be triggers - state changed, event happened - and don't necessarily need to be expressed as a metric

## Technical References
- https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-checkpointing-and-replay
- https://medium.com/@tsuyoshiushio/durable-functions-101-35aa3919f182
- https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview
- https://docs.microsoft.com/en-gb/rest/api/monitor/metricalerts/createorupdate
- https://docs.microsoft.com/en-us/azure/azure-monitor/platform/alerts-webhooks#configure-webhooks-via-the-azure-portal

[Azure Functions]: https://docs.microsoft.com/en-us/azure/azure-functions/functions-overview
[Durable Functions]: https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview