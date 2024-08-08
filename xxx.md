fields @timestamp, @message
| filter @message like /callApi/
| parse @message "callApi code=* url=* type=* used *" as statusCode, url, type, usedTime
| filter url == "xxx/xxx/xxxxxx"
| stats count(*) by bin(1m)
