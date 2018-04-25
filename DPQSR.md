**1.  Purchase Funnel Starts**

1a. All Purchase Funnel Starts
```
SELECT device_code, date_trunc('month', event_timestamp) as month, count (*) as views 
FROM sp_telegraph.ui
WHERE event_timestamp >= '2018-01-01'
AND target LIKE 'PAYWALL_CTA'   
AND product_code = 'hboNow'
GROUP BY 1,2
ORDER BY 1,2 ASC
```

1b. Distinct Purchase Funnel Starts (dedups if someone clicks on CTA twice in a session)
```
SELECT device_code, date_trunc('month', event_date) as month, count(session_id)
FROM
(SELECT device_code, date(event_timestamp) as event_date, session_id
FROM sp_telegraph.ui
WHERE event_timestamp >= '2018-01-01'
AND target LIKE 'PAYWALL_CTA'   
AND product_code = 'hboNow'
GROUP BY 1,2,3
ORDER BY 1,2,3 ASC)
GROUP BY 1,2
```

Some Notes: 
-- assumes that all these complete successfully for now which is unlikely. better to base this off of actual people who reach next page but need to verify the first page is the Register screen for all devices. Is it? Not sure.
-- available in Telegraph for Android (tablet + mobile), desktop, ios (phone + tablet), uwp_tablet (tho minimal data there)
-- not available for Roku, Samsung or Apple TV
-- I thought there was some data for Apple TV in Telegraph so will need to investigate why nothing for this is coming up, perhaps the instrumentation on the Call to Action button is different?

**2.       Purchase Funnel Conversion Rate**

```

```


**3.       Purchase Funnel Completion Tim**
**4.       Sign-In Starts**
**5.       Content Consumption Rate**
**6.       Discovery Time**
**7.       Diversity of Content Consumed**
**8.       Session Time**
**9.       Plays & Extended Plays**

