**1.  Purchase Funnel Starts**

1a. All Purchase Funnel Starts (accounts for diff implementation on mobile/ ctv devices, but not session deduped)
```
SELECT device_code, date_trunc('month', event_timestamp) as month, count (*) as views 
FROM sp_telegraph.ui
WHERE event_timestamp >= '2018-01-01'
AND target LIKE 'PAYWALL_CTA'   
AND product_code = 'hboNow'
GROUP BY 1,2
union all
SELECT device_code, date_trunc('month', event_timestamp) as month, count (*) as views 
FROM sp_telegraph.navigation
WHERE event_timestamp >= '2018-01-01'
AND view_type = 'Activation'  
AND product_code = 'hboNow'
GROUP BY 1,2
```

1b. Distinct Purchase Funnel Starts (deduped and accounts for different implementation on mobile/ ctv devices)
```
-- this part addresses the mobile apps
SELECT device_code, date_trunc('month', event_date) as month, count(session_id)
FROM
(SELECT device_code, date(event_timestamp) as event_date, session_id
FROM sp_telegraph.ui
WHERE event_timestamp >= '2018-01-01'
AND target LIKE 'PAYWALL_CTA'   
AND product_code = 'hboNow'
GROUP BY 1,2,3)
GROUP BY 1,2
union all
-- this part addresses the ctv apps (different implementation)
SELECT device_code, date_trunc('month', event_date) as month, count(session_id)
FROM
(SELECT device_code, date(event_timestamp) as event_date, session_id
FROM sp_telegraph.navigation
WHERE event_timestamp >= '2018-01-01'
AND view_type = 'Activation'  
AND product_code = 'hboNow'
GROUP BY 1,2,3)
GROUP BY 1,2
```

Some Notes: 

- assumes that all these complete successfully for now which is unlikely. better to base this off of actual people who reach next page but need to verify the first page is the Register screen for all devices. Is it? Not sure.
- available in Telegraph for Android (tablet + mobile), desktop, ios (phone + tablet), uwp_tablet (tho minimal data there)
- not available for Roku, Samsung or Apple TV and not sure Samsung includes all Tizen models
- I thought there was some data for Apple TV in Telegraph so will need to investigate why nothing for this is coming up, perhaps the instrumentation on the Call to Action button is different? Asked Carlos and he said that currently the CTA button is not tagged in those (just mobile) so to pull for those others use navigation  and look for view type "Activation" which is a view to activation page. 



**2.       Purchase Funnel Conversion Rate**
Not sure that this purchase successful event corresponds to the subscribe event that is being worked on as part of receipts processing but for the purposes of understanding funnel conversion rate, let's use this now with a huge caveat that this may change once transaction work fully completes.

2a. Monthly transactions by Device
```
SELECT device_code, date_trunc('month', event_timestamp) as month, count (*) as purchase_successful
FROM sp_telegraph.purchase
WHERE event_timestamp >= '2018-01-01'
and product_code = 'hboNow'
GROUP BY 1,2
ORDER BY 1,2 ASC
```
2b. Purchase Funnel Conversion Rate 
```

```

**3.       Purchase Funnel Completion Time**
Same dependency as 2

```
tbd
```

**4.       Sign-In Starts**
Definition in DP Technical doc: User signs into the app. This event also gets triggered during user token authentication checks. Not sure if there is a way to separate out just active sign ins.

```
SELECT device_code, date_trunc('month', event_timestamp) as month, count (*) as signins
FROM sp_telegraph.user
WHERE event_timestamp >= '2018-01-01'
AND event = 'user:signin' 
and product_code = 'hboNow'
GROUP BY 1,2
ORDER BY 2,1 ASC
```

**5.       Content Consumption Rate**
Requires more discussion and context

```
tbd
```

**6.       Discovery Time**
Time to first play

**7.       Diversity of Content Consumed**
Requires more discussion and context

```
tbd
```

**8.       Session Time**
A bit tricky the way the data is structured (requires collapsing all tables to derive session time. Will try to figure out an efficient way of running this.

**9.       Plays & Extended Plays**
Will rely on Conviva for now to provide Streams but ideally we'd use Telegraph in the future. 
What is extended plays?

