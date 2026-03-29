

filter idSource = "IVR" | stats count() as totalOccurrences by trackingId | filter totalOccurrences > 1 | sort totalOccurrences desc

filter msg in ["METRICS:login-success", "METRICS:login-failed"] 
| filter `x-source-app` != "bot" | filter `x-source-app` != "cookie" | filter idSource != "IVR" 
| stats sum(ispresent(reqId) and idType = "qid") as qid, 
        sum(ispresent(reqId) and idType = "did") as did, 
        sum(!ispresent(reqId)) as webauthn 
        by msg


filter msg = "METRICS:login-success"
| filter `x-source-app` != "bot"
| filter idSource != "IVR"
| filter (idType = "DID" or idType = "QID")
| fields if(extra.authenticatorType = "webAuthn" or authenticatorType = "webAuthn", "WebAuthn", if(idType = "QID", "QR Code", "Type Code")) as display_name
| stats count(reqId) as count by display_name
| sort count desc


filter msg = "authentication attempted" or msg = "user authenticated" or msg = "ivr validated"
| stats 
    min(@timestamp) as startTime, 
    max(@timestamp) as endTime,
    count_distinct(msg) as stepsReached,
    earliest(notification_method) as method,
    earliest(countrycode) as country
  by trackingId
| filter stepsReached = 3
| fields (endTime - startTime) / 1000 as totalTimeSeconds
| stats 
    avg(totalTimeSeconds) as averageDuration,
    pct(totalTimeSeconds, 90) as p90Duration,
    count() as totalUsers
  by country, method
| sort p90Duration desc

filter idSource = "IVR"
| filter msg = "METRICS:login-success"
| stats 
    count() as totalOccurrences, 
    min(@timestamp) as firstOccurrence, 
    max(@timestamp) as latestOccurrence
  by trackingId
| filter totalOccurrences > 1
| fields 
    (latestOccurrence - firstOccurrence) / 1000 as gapSeconds,
    latestOccurrence as Latest_Timestamp,
    trackingId as Duplicate_ID
| sort latestOccurrence desc

fields reqId, msg, `extra.headers.x-forwarded-for` as xff
| filter msg in ['logMetricsMiddleware:newRequest', 'SCIM_CREATE START', 'SCIM_GET_START']
| parse xff "*,*" as firstIp, rest
| stats
    latest(if(msg='logMetricsMiddleware:newRequest',
              if(ispresent(firstIp), firstIp, xff),
              null)) as srcIp,
    max(if(msg='SCIM_CREATE START', 1, 0)) as hasCreate,
    max(if(msg='SCIM_GET_START', 1, 0)) as hasGet
  by reqId
| filter hasCreate = 1 or hasGet = 1
| stats count() as Count
  by srcIp,
     if(hasCreate = 1, 'SCIM_CREATE', 'SCIM_GET') as scimType
| sort Count desc