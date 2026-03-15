**CRONTAB: Automated Task Scheduling**

Crontab is a Linux utility used to schedule tasks to run automatically at specified intervals. It can execute any command or script that runs in Bash, including Python, Node.js, PHP and system commands.



Key Locations and Commands

\* crontab -e: Edit the current user's cron jobs.

\* crontab -l -u \[user]: View another user's cron jobs (requires root access).

\* /etc/crontab: Contains system-wide cron jobs.



System Directories: Scripts can also be placed in specific directories for automatic execution:

\* /etc/cron.hourly/

\* /etc/cron.daily/

\* /etc/cron.weekly/

\* /etc/cron.monthly/



Crontab Syntax

The format for a crontab entry follows a specific sequence of time and user fields.



MIN HOUR DAYOFTHEMONTH MONTH DAYOFTHEWEEK USER COMMAND



Here are some examples:

Daily: 5 4 \* \* \* root test.sh (Runs the test.sh file by the root user every day at 4:05)

Weekly: 5 16 \* \* 1 root test.sh (Runs the test.sh file by the root user every Monday at 16:05 (4:05 PM)

Monthly: 5 4 1 \* \* root test.sh (Runs on the 1st of every month at 04:05)

Interval: 0/2 \* \* \* \* root test.sh (Runs every 2 minutes)

Specific Date: 5 4 1 8 \* root test.sh (Runs on August 1st at 04:05)

