
 **Fields to query**

amznApiGatewayId amznReqId reqId appId orgCode metric metricName
 
 **First Log for Web Requests**

```
filter msg = "logMetricsMiddleware:newRequest"
```

**IVR Validations vs auths**

```
#filter msg = 'METRICS:ivr-code-validated'
filter msg = 'METRICS:login-success'
| filter idSource = 'IVR'
| stats count()
```

**IVR Generated vs Success**

```
fields @timestamp, @message
#| filter (strcontains("__orgcode", "ALL") or orgCode = "__orgcode")
| filter idSource != "IVR"
#| filter (strcontains("__appid", "ALL") or appId = "__appid")
| filter orgCode = "EFX"
| stats 
    sum(msg = "METRICS:ivr-code-generated") as CodesGeneratedCount,
    sum(msg = "METRICS:login-success" and idSource = "IVR") as CodesValidatedCount
```

IVR Success by Authenticator

```
filter idSource = "IVR"
| filter (msg = "METRICS:login-success")
| filter (strcontains("ALL", "ALL") or orgCode = "ALL")
| stats count() by x-source-app as Authenticator
| sort count desc
```

Login Attempts

```
filter msg = "METRICS:verifyId-process"
| filter (strcontains("__orgcode", "ALL") or orgCode = "__orgcode")
| filter idSource != "IVR"
| filter (strcontains("__appid", "ALL") or appId = "__appid")
| stats count() as Count by x-source-app as AuthType
| sort Count desc
```

Login Success & Failures by Method

```
filter msg in "METRICS:login-success", "METRICS:login-failed"
| filter x-source-app != "bot"
| filter x-source-app != "cookie"
| filter idSource != "IVR"
| stats 
    sum(ispresent(reqId) and idType = "QID") as qid,
    sum(ispresent(reqId) and idType = "DID") as did,
    sum(!ispresent(reqId)) as webauthn
by msg
```

Login Success by cookie vs bot vs human

```
filter msg = "METRICS:login-success"
#| filter (strcontains("__orgcode", "ALL") or orgCode = "__orgcode")
| filter idSource != "IVR"
#| filter (strcontains("__appid", "ALL") or appId = "__appid")
| fields Source = if(`x-source-app` = "bot", "bot", if(`x-source-app` = "cookie", "cookie","Human Scan/Type/Passkey"))
| stats count() as Count by Source
```
Login Success by x-source-app and authenticatorPlatform

```
filter msg = "METRICS:login-success"
| filter `x-source-app` != 'cookie'
| filter idSource != "IVR"
| stats count() by `x-source-app`,authenticatorPlatform
```