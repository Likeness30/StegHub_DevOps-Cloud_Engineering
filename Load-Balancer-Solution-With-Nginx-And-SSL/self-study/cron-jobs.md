# Cron Jobs
Cron is a tool in Unix-like systems that allows you to schedule tasks to run automatically at specific times. This guide explains how Cron works, its basic syntax, and common uses.

## Basic Concepts
What is Cron?
Cron is a program that runs in the background and automatically executes commands or scripts at set times. It’s useful for automating tasks like backups, updates, and system monitoring.

## Crontab
Crontab (Cron table) is a file where you define the jobs you want to schedule. Each user on the system has their own crontab file. You can edit your crontab by running:
```
crontab -e
```
## The format for scheduling a job looks like this:

```
* * * * * /path/to/command
```
## Here’s what each part means:
Minute (0 - 59): At what minute the job will run.
Hour (0 - 23): At what hour the job will run.
Day of the month (1 - 31): On which day of the month.
Month (1 - 12): In which month.
Day of the week (0 - 7): On which day of the week (0 and 7 are Sunday).

## Common Examples
Run a job every 5 minutes:
```
*/5 * * * * /path/to/job.sh
```
## Managing Cron Jobs
List Cron Jobs
To see all your scheduled jobs, use:
```
crontab -l
```
## Edit Cron Jobs
To edit your cron jobs, use:
```
crontab -e
```
## Remove Cron Jobs
To delete all your cron jobs:
```
crontab -r
```
