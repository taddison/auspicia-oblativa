# Auspicia Oblativa
React to the portents of the gods.  Or monitoring systems.

## What is it?
A sink for notifications from monitoring systems that can triage them and then dispatch notifications to other systems (Slack, MS Teams, PagerDuty).

## Why does it exist?
Configure your alerts to land in a single endpoint and then as you iterate on the downstream systems you don't need to modify your alert rules.  If you want to add fan-out, ignores, or some really fancy remediation you can do that.

## How can I run it?
You can't, it doesn't exist yet.

You can find details on what is planned in the roadmap file in the docs folder.

Inspiration for this system comes from the code you can find in the [OMS to Slack][OMS to Slack repo] repo.

[OMS to Slack repo]: https://github.com/taddison/blog-oms-to-slack