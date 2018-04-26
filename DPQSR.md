**1.  Purchase Funnel Starts**

* 1a. All Purchase Funnel Starts (accounts for diff implementation on mobile/ ctv devices, but not session deduped)
  ```
  SELECT device_code, date_trunc('month', event_timestamp) as month, count (*) as views 
  FROM telegraph.ui
  WHERE dt >= '2018-01-01'
  AND target LIKE 'PAYWALL_CTA'   
  AND product_code = 'hboNow'
  GROUP BY 1,2
  union all
  SELECT device_code, date_trunc('month', event_timestamp) as month, count (*) as views 
  FROM telegraph.navigation
  WHERE dt >= '2018-01-01'
  AND view_type = 'Activation'  
  AND product_code = 'hboNow'
  GROUP BY 1,2
  ```

* 1b. Distinct Purchase Funnel Starts (deduped and accounts for different implementation on mobile/ ctv devices)
  ```
  -- this part addresses the mobile apps
  SELECT device_code, date_trunc('month', event_date) as month, count(session_id) as starts
  FROM
  (SELECT device_code, date(event_timestamp) as event_date, session_id
  FROM telegraph.ui
  WHERE dt >= '2018-01-01'
  AND target LIKE 'PAYWALL_CTA'   
  AND product_code = 'hboNow'
  GROUP BY 1,2,3)
  GROUP BY 1,2
  union all
  -- this part addresses the ctv apps (different implementation)
  SELECT device_code, date_trunc('month', event_date) as month, count(session_id) as starts
  FROM
  (SELECT device_code, date(event_timestamp) as event_date, session_id
  FROM telegraph.navigation
  WHERE dt >= '2018-01-01'
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
* Not sure that this purchase successful event corresponds to the subscribe event that is being worked on as part of receipts processing but for the purposes of understanding funnel conversion rate, let's use this now with a huge caveat that this may change once transaction work fully completes.

* 2a. Monthly transactions by Device
  ```
  SELECT device_code, date_trunc('month', event_timestamp) as month, count (*) as purchase_successful
  FROM sp_telegraph.purchase
  WHERE event_timestamp >= '2018-01-01'
  and product_code = 'hboNow'
  GROUP BY 1,2
  ORDER BY 1,2 ASC
  ```
  * 2b. Purchase Funnel Conversion Rate 
  ```
  Will complete tomorrow (4/26)
  ```

**3.       Purchase Funnel Completion Time**
Same dependency as 2; also need clarity around timestamps given anomolies encountered in #6

**4.       Sign-In Starts**
* Definition in DP Technical doc: User signs into the app. This event also gets triggered during user token authentication checks. Not sure if there is a way to separate out just active sign ins. (
* Using target=login from ui table provides some request to login data for desktop; largely not helpful)

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
* Requires more discussion and context

**6.       Discovery Time**
* Time to first play
* there seems to be inconsistency in timestamps that is causing anomolous data to be reported. I think this is because the way Telegraph keeps a session is not necessarily standard; sessions don't seem to end when there is no period of inactivit. I've done Median to get rid of those extremes. The Median window function won't work in Athena; need to run from Redshift.

```
select month, device_code, med_time_to_stream
from
(SELECT
     month,
     device_code,
     median(time_to_stream) OVER (PARTITION BY month, device_code) as med_time_to_stream
	from 
	(select a.session_id, a.device_code, date_trunc('month', event_timestamp) as month, a.event_timestamp, b.first_stream_time
	,(case when b.first_stream_time is not null then 1 else 0 end) as stream_flag
	,(case when b.first_stream_time is not null then date_diff('second', a.event_timestamp, b.first_stream_time) else null end) as time_to_stream
from sp_telegraph.application a
left join
(select session_id, device_code, min(timestamp) as first_stream_time from sp_telegraph.video where event = 'video:start' and dt >= '2018-01-01' group by 1,2) b
on a.session_id = b.session_id
where a.event = 'application:launch' and a.dt >= '2018-01-01') a ) b
group by 1,2,3
```

**7.       Diversity of Content Consumed**
* Requires more discussion and context

**8.       Session Time**
* A bit tricky the way the data is structured (requires collapsing all tables to derive session time. Will try to figure out an efficient way of running this.
* pausing on this until we get more clarity on timestamps of events

**9.       Plays & Extended Plays**
* Will rely on Conviva for now to provide Streams but ideally we'd use Telegraph in the future. 
* What is extended plays?

 ```
    select date_trunc('month', a.start_time_utc) as month, count(a.conviva_session_id) as stream_count
     from 
     (select hbo_uuid,device_os,startup_time_ms,ip_address,conviva_session_id,start_time_utc, playing_time_ms 
     from conviva.hbogo_streaming a, identity.idlinktimeseries b
     where (case when substring(viewerid,1,5)='mlbam' then substring(viewerid,7,200) else viewerid end)=b.external_id  and b.dt='2018-04-26' 
     and upper(session_tag_affiliate)='HBO_OTT' and date(start_time_utc)>=date('2018-01-01') and date(start_time_utc)<date('2018-04-26') 
     union -- usage from now coviva
     select hbo_uuid,device_os,startup_time_ms,ip_address,conviva_session_id,start_time_utc, playing_time_ms
     from conviva.hbonow_streaming a, identity.idlinktimeseries b
     where (case when substring(viewerid,1,5)='mlbam' then substring(viewerid,7,200) else viewerid end)=b.external_id  and b.dt='2018-04-26' 
     and  date(start_time_utc)>=date('2018-01-01') and date(start_time_utc)<date('2018-04-26')) a
     group by 1)
  ```
