#**1. -- Purchase Funnel Starts**

```
SELECT device_code, (event_timestamp) as month, count (*) as views 
FROM sp_telegraph.ui
WHERE event_timestamp >= '2018-01-01'
AND target LIKE 'PAYWALL_CTA'   
and product_code = 'hboNow'
GROUP BY 1,2
ORDER BY 1,2 ASC
```

Some Notes: 
-- assumes that all these complete successfully for now which is unlikely. better to base this off of actual people who reach next page but need to verify the first page is the Register screen for all devices. Is it? Not sure.
-- available in Telegraph fpr Android (tablet + mobile), desktop, ios (phone + tablet), uwp_tablet (tho minimal data there)
-- not available for Roku, Samsung or Apple TV
-- I thought there was some data for Apple TV in telegraph so will need to investigate why nothing for this is coming up, perhaps a different implementation

#2.       Purchase Funnel Conversion Rate
#3.       Purchase Funnel Completion Tim
#4.       Sign-In Starts
#5.       Content Consumption Rate
#6.       Discovery Time
#7.       Diversity of Content Consumed
#8.       Session Time
#9.       Plays & Extended Plays

