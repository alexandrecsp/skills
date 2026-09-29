# Triage Labels

The skills speak in terms of five canonical triage roles. This file maps those roles to the `Status:` strings used in this repo's ticket files (see `issue-tracker.md`).

| Canonical role    | `Status:` string  | Meaning                                  |
| ----------------- | ----------------- | ---------------------------------------- |
| `needs-triage`    | `needs-triage`    | Maintainer needs to evaluate this ticket |
| `needs-info`      | `needs-info`      | Waiting on more information              |
| `ready-for-agent` | `ready-for-agent` | Fully specified, ready for an AFK agent  |
| `ready-for-human` | `ready-for-human` | Requires human implementation            |
| `wontfix`         | `wontfix`         | Will not be actioned                     |

When a skill mentions a role (e.g. "apply the AFK-ready triage label"), set `Status:` to the corresponding string from this table. The two category roles, `bug` and `enhancement`, go in the `Category:` line and are not configurable.

Edit the right-hand column to match whatever vocabulary you actually use.
