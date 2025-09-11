# PostgreSQL on Linux

I installed PostgreSQL on my Ubuntu Linux server and stumbled upon some problems having to do with access and permissions. I decided to figure things out and in this blog post I'll try to describe not how you get things done, but how you can inspect the state of the PostgreSQL server, its contents and the status and privileges of its users.

## Checking if PostgreSQL runs

Run this in the Ubuntu shell as root or other user.

- `systemctl status` gives list of systemd units. There should be postgres stuff in it.
- `systemctl status postgresql` provides the following if postgres runs well:

```
● postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/usr/lib/systemd/system/postgresql.service; enabled; preset: enabled)
     Active: active (exited) since Sat 2025-09-06 19:18:55 UTC; 5 days ago
   Main PID: 894 (code=exited, status=0/SUCCESS)
        CPU: 4ms
```

The first 'enabled' tells you it works, the second that postgres will be started automatically on server reboot. The 'active' word means postgres is running fine.

