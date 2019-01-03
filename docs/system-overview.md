# AO System Overview

The system is built as a set of [Azure Functions].  The functions chain using [Durable Functions] rather than wiring up the intermedia storage queues.

The system reacts to *alerts* triggered by external systems.  These are ingested by calling an endpoint, and are most probably webhooks.  The flow of an alert is typically:

- Monitoring sink endpoint called
- Monitor payload processed and turned into a standard `MonitorNotification` object
- `AlertRules` are evaluated to determine if the monitor should fire an `Alert`
- If an `Alert` is fired then one or more `Notifications` are triggered
- Each `Notification` is processed, and zero or more outputs are generated

>A notification might generate zero outputs if some kind of flow control/dedupe/mute is in place

## AlertRule
Alert rules are comprised of (mandatory):

- A name for the incoming monitor notification
- A notification target

Optional:
- Criteria to evaluate the monitor (no criteria means always true)
- A filter for the dimensions of the monitor (e.g. server name)

An incoming monitor (e.g. `Server-CPU`) can have multiple alerts attached to it.  This might look like this:

```json
{
    "AlertRules": [
        {   
            "AlertRuleId": "Server CPU - All Servers",
            "MonitorName": "Server-CPU", 
            "Notifications": [{
                "NotificationTargetId": "Slack",
                "Channel": "#proudctionisburning"
            }]
        }
        ,{  
            "AlertRuleId": "Server CPU - Web Servers",
            "MonitorName": "Server-CPU",
            "Conditions": [
                { "ServerName": "web" }
            ],
            "Notifications": [{
                "NotificationTargetId": "Slack",
                "Channel": "#webteam",
                "Message": "Web CPU is high"
            }]
        }
    ]
}
```

## Notification

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
  - Because alert rule notifications are so flexible (maybe the alert doesn't even have a metric, it might go to an SMS, Webhook JSON payload, Slack, MS Teams channel, PagerDuty), testing definitions is important (both when rules are changed, and code that handles them is updated)
- What does testing in production look like?

## Answered questions
- What does local testing look like?
  - Durable function testing works fine with the storage emulator (verified with functions 1.0.24 and DurableTask 1.7.0).  Requires an explicit reference to Newtonsoft.Json 11.0.2.
- Notification configuration split
  - `NotificationTarget` contains configuration data - e.g. for MS Teams - Id, Endpoint Uri
  - `AlertRule` contains details to be sent to the `NotificationTarget`, e.g. Message, CardType
  - Limited replacement tokens (e.g. METRIC_AVG, METRIC_MIN_MAX_AVG)
- If a new orchestrator function is deployed the hub name needs to be changed so that new executions will get dedicated replays

## Technical References
- https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-checkpointing-and-replay
- https://medium.com/@tsuyoshiushio/durable-functions-101-35aa3919f182
- https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview

[Azure Functions]: https://docs.microsoft.com/en-us/azure/azure-functions/functions-overview
[Durable Functions]: https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview