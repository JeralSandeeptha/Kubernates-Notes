# Cron Job
- A `CronJob` allows you to run Jobs on a time-based schedule (just like UNIX cron)
- It creates a Job at scheduled intervals, and each Job runs independently

| Feature             | Description                             |
| ------------------- | --------------------------------------- |
| Time-based trigger  | Uses cron format (e.g., `*/5 * * * *`)  |
| Job template        | Uses the same spec as a normal Job      |
| Independent runs    | Each execution is its own Job           |
| Missed run handling | Can skip or catch up on missed runs     |
| History tracking    | Keeps logs of completed and failed jobs |
