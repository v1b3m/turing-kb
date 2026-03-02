Local time: 14:28 UTC + 3

Value: 5

![[Screenshot from 2026-02-28 14-28-02.png]]

Value: 8

![[Screenshot from 2026-02-28 14-28-12.png]]

Value: 11

![[Screenshot from 2026-02-28 14-28-21.png]]

Value: 15

![[Screenshot from 2026-02-28 14-28-28.png]]



## Outcome

Fixed the Updated column breadcrumb to show current time in UTC-8 (America/Los_Angeles) using `toLocaleString('sv-SE', { timeZone: 'America/Los_Angeles' })` instead of subtracting hours from local time. The input value no longer affects the displayed timestamp, matching original ServiceNow behavior.

Commit: 5df53e1