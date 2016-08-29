===================
this article explains how to configure the following settings in Hive:
====================

> ###hive.server2.session.check.interval

> ###hive.server2.idle.operation.timeout

> ###hive.server2.idle.session.timeout



####1). hive.server2.idle.session.timeout

Session will be closed when not accessed for this duration of time, in milliseconds; disable by setting to zero or a negative value.
For example, the value of “86400000” indicate that the session will be timed out after 1 day of inactivity.

####2). hive.server2.session.check.interval
The check interval for session/operation timeout, in milliseconds, which can be disabled by setting to zero or a negative value.
For example, the value of “3600000” indicate that the session will be checked every 1 hour.

####3) hive.server2.idle.operation.timeout
Operation will be closed when not accessed for this duration of time, in milliseconds; disable by setting to zero. For a positive value, checked for operations in terminal state only (FINISHED, CANCELED, CLOSED, ERROR). For a negative value, checked for all of the operations regardless of state.
For example, the value of “7200000” indicate that the query/operation will be timed out after 2 hours if it is still running.

So if you combine the three settings from above examples, we can summarize the following use cases:

1) If you started a HS2 session, beeline for example, and without doing anything afterwards, HS2 will trigger 24 session checking before it determines that 24 hours has passed since last activity, then session will be closed

2) If you worked on beeline for 2 hours and then leave the beeline open without doing anything afterwards, HS2 will trigger total of 26 session checking (2 while you worked and another 24 while in idle), and the session will be closed 26 hours after initially opened.

3) If you worked on beeline for 2 hours, and you started running a query that will run for 1 hour and then returns result, the idle timer actually starts from the time when data returns, so if you don’t do anything afterwards, HS2 will kill the session after another 24 hours, so in total, the session lasted 27 hours (2+1+24)

4) If, for instance, your query takes longer than the 2 hours defined by hive.server2.idle.operation.timeout, then the query will be cancelled, hence you will lose the result all together

5) If hive.server2.session.check.interval = 0 and hive.server2.idle.session.timeout > 0, then it will have the same effect that hive.server2.idle.session.timeout = 0, because no idle timer will be triggered as check interval is disabled

6) If hive.server2.session.check.interval > hive.server2.idle.session.timeout > 0, then the actual time out will be the same as hive.server2.session.check.interval.

For example if hive.server2.session.check.interval = 20 minutes, but hive.server2.idle.session.timeout = 10 minutes, then timeout will happen when checking happens, which is 20 minutes.

The basic rule of thumb is as follows:
hive.server2.session.check.interval < hive.server2.idle.operation.timeout < hive.server2.idle.session.timeout

**And the recommended values are:**

```html
hive.server2.session.check.interval = 1 hour

hive.server2.idle.operation.timeout = 1 day

hive.server2.idle.session.timeout = 3 days
```

This will work for most of clusters, but you can change the value depending on how your cluster behaves and how long each query run for.
