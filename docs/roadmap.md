# Auspicia Oblativa Roadmap

Webhooks in.  Webhooks out.  Designed to take notifications from monitoring systems and apply filters and routing rules to let you know something needs your attention (Slack, MS Teams, PagerDuty).

## 1.0
- Provide a single set of endpoints to sink Azure Monitor alert notifications with custom output
  - New alert
- Provide mapping and filtering of alerts
  - Ignore notification if some condition is not met (e.g. if server = x and metric >= y)
  - Route to a specified target (e.g. channel in Slack)
- Provide notifications to Slack, MS Teams

## Later/Maybe
- Logging and inspection of all alert payloads received
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