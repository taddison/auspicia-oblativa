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

An incoming notification name (e.g. `Server-CPU`) can have multiple alerts attached to it.  This might look like this:

```json
{
    "AlertRules": [
        {   
            "MonitorName": "Server-CPU", 
            "Notification": "Slack-AllProd"
        }
        ,{  
            "MonitorName": "Server-CPU",
            "Conditions": [
                { "ServerName": "web" }
            ],
            "Notification": "Slack-WebTeam" 
        }
    ]
}
```

This alert configuration would send a Slack message to the `Slack-AllProd` channel for every CPU notification received, and also to the `Slack-WebTeam` channel only when the ServerName contains web.

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
- How should notifications be configured?  Configuring some of the notification in the alert rule allows for customised error messages, but it means any complex processing (e.g constructing a detailed card to send via MS Teams) is much harder.  Maybe a fairly small set of defined options for each notification?
  - Should there be tokens/token replacement in the messages?  Notifications should be easy to understand
- Some notification configuration should be kept out of the alert rule, so that changes to e.g. an api key don't require updating alert rules.
- What does testing look like?  Both in terms of testing the config isn't broken (a bad deploy of a config renders the app useless), and in terms of any of the individual functions work.
- What happens if an app is deployed during function orchestration - how do existing contexts that might be executing transition?
- What does testing in production look like?

[Azure Functions]: https://docs.microsoft.com/en-us/azure/azure-functions/functions-overview
[Durable Functions]: https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview