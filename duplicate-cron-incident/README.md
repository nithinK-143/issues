# Incident: Duplicate Order Cancellation

## What Happened  
Order **L2601001700** was cancelled twice by the system causing duplicate stock return & mails

---

## Evidence  

Two different PM2 process logs show the same order cancelled at the same time:

#### - LIVE-v4-3-5-core-crons-out-9.log
![Log showing first successful cancellation](https://screenrec.com/share/bLOGfYqKxE)

```
Cancelling order L2601001700 placed at Fri Jan 30 2026 01:29:25 GMT+0530, should cancel by Sat Jan 31 2026 01:29:25 GMT+0530 cancelled on Sat Jan 31 2026 02:00:00 GMT+0530 (India Standard Time)
Successfully cancelled order L2601001700
```

#### - LIVE-v4-3-5-core-crons-out-13.log
![Log showing second successful cancellation](https://github.com/user-attachments/assets/c738b502-09b1-41ad-a2f5-15016108d3e1)
```
Cancelling order L2601001700 placed at Fri Jan 30 2026 01:29:25 GMT+0530, should cancel by Sat Jan 31 2026 01:29:25 GMT+0530 cancelled on Sat Jan 31 2026 02:00:00 GMT+0530 (India Standard Time)
Successfully cancelled order L2601001700
```

This confirms two processes performed the same action.

---

## Cause  

A restart caused **two instances of the cron process** to run at the same time.

---

## Suggested Fixes

**1. Guard the cancellation update**

Only cancel an order if it is not already cancelled.  
If another process already cancelled it, stop and do nothing else.

This prevents duplicate stock return and duplicate emails.

---

**2. Restart PM2 safely**

When restarting:
- Stop the cron process fully before starting it again  
- Confirm only one instance is running after restart  
- Avoid rapid or repeated restarts  

This reduces the chance of two cron processes running at the same time.

---

**3. Use controlled server restarts during updates**

During deployments or major updates:
- Stop the application before making changes  
- Or restart the Windows VM cleanly instead of restarting processes individually  
- Ensure the system comes back up with only one cron instance running  

Prevents old and new processes from running at the same time.
```