**views on paywall screen for last 30 days ; note this is not distinct sessions. A user may have viewed New Sub Paywall multiple times
```
SELECT date_trunc('day', event_timestamp) as day, count (*) as views 
FROM sp_telegraph.navigation
WHERE event_timestamp > date_trunc('month', (getdate())::date - interval '30 days') 
AND view_type LIKE 'New Subscriber Paywall'
and device_code in ('android', 'andrd_tblt')    -- update device
and product_code = 'hboNow'
GROUP BY 1
ORDER BY 1 ASC
```

**views on create profile screen (register)
```
SELECT date_trunc('day', event_timestamp) as day, count (*) as views 
FROM sp_telegraph.navigation
WHERE event_timestamp > date_trunc('month', (getdate())::date - interval '30 days') 
AND view_type LIKE 'Register'
and device_code in ('android', 'andrd_tblt')    -- update device
and product_code = 'hboNow'
GROUP BY 1
ORDER BY 1 ASC
```

**register events
```
SELECT device_code, date_trunc('month', event_date) as month, count(session_id)
FROM 
(SELECT device_code, date(event_timestamp) as event_date, session_id
FROM sp_telegraph.ui
WHERE event_timestamp >= '2018-01-01'
AND target LIKE 'PROFILE_REGISTERED'   
AND product_code = 'hboNow'
GROUP BY 1,2,3
ORDER BY 1,2,3 ASC)
GROUP BY 1,2
```

**purchases (but not fully working on DAP, do not use right now)
```
SELECT date_trunc('day', event_timestamp) as day, count (*) as purchases,
      FROM sp_telegraph.purchase
      WHERE event_timestamp > date_trunc('month', (getdate())::date - interval '30 days') 
      and device_code in ('android', 'andrd_tblt')    -- update device
		  and product_code = 'hboNow'
      GROUP BY day
      ORDER BY day ASC
```

