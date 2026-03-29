


{"level":30,"time":1772829700523,"pid":126692,"hostname":"DESKTOP-3QUCA0P","reqId":"req-1","req":{"method":"POST","url":"/oauth/token","host":"localhost:5005","remoteAddress":"127.0.0.1","remotePort":1071},"msg":"incoming request"}
host: localhost:5005

{"level":50,"time":1772945882067,"pid":35212,"hostname":"DESKTOP-3QUCA0P","type":"FASTIFY_ERROR","error":{"code":"FST_ERR_CTP_INVALID_JSON_BODY","name":"FastifyError","statusCode":400,"message":"Body is not valid JSON but content-type is set to 'application/json'"},"request":{"route":{"method":["POST"],"url":"/oauth/token","bodyLimit":1048576,"handlerTimeout":0,"attachValidation":false,"logLevel":"","exposeHeadRoute":true,"prefixTrailingSlash":"both","config":{"url":"/oauth/token","method":["POST"]},"schema":{"consumes":["application/json","application/x-www-form-urlencoded"],"body":{"type":"object","required":["grant_type"],"properties":{"grant_type":{"type":"string"},"assertion":{"type":"string"},"audience":{"type":"string"},"scope":{"type":"string"},"issuer":{"type":"string"}}}}}}}