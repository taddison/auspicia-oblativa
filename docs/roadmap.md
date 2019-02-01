# Auspicia Oblativa Roadmap

Webhooks in.  Webhooks out.  Designed to take notifications from monitoring systems and apply filters and routing rules to let you know something needs your attention.  Targeting Azure Monitor -> MS Teams for v1.

Some guiding principles:

- Authoring new alerts should allow for easy testing (does it fire when I think? does it look right? have I broken the world?)
- Rich message delivery should be easy to author (server X triggers an alert? we have a custom dashboard for that) - aim to leverage the message card/adaptive card format
- Initial deploy should support stateless routing, with an eye to extend later with throttling, suppression, status, and more.

---

## Old stuff

No longer part of the roadmap - kept for inspiration pending cleanup

---

## 0.1
- Schema for a `InNotification`, `NotificationRule`, `OutNotification`, `OutNotificationTarget`
- Implementation of the logging target
- Implementation of a trigger monitor (call endpoint, generate a MonitorNotification)
- Test that supports the trigger monitor to route to the log target
  - Hardcoded match-everything and route to log NotificationRule

## 1.0
- Provide a single set of endpoints to sink Azure Monitor notifications with custom output
  - New alert
- Provide mapping and filtering of notifications
  - Ignore notification if some condition is not met (e.g. if server = x and metric >= y)
  - Route to a specified target (e.g. channel in Slack)
- Provide notifications to Slack, MS Teams

## Later/Maybe
- Logging and inspection of all notification payloads received
  - Payload replay
- Tracer tokens/test (check things work)
- Routing rule rollback/deployment via means other than a deploy
- Track alert status
  - Receive updates
- Have a monitoring component
  - Execute queries on targets and raise alerts if required
  - Execute queries for existing alerts
- Two-way PD integration
- State management of alerts
- Suppressions
- Alert rollups
- Dedupe