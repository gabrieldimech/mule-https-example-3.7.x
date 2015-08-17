# mule-https-example-3.7.x

This page describes how to define an HTTPS client and server flow in Mule 3.7. This example shows how to configure one-way SSL only.

The following is an HTTPS listener for the server flow:

```xml
<http:listener-config name="httpsServerConnector"
	host="localhost" port="8083" doc:name="HTTP Listener Configuration"
	protocol="HTTPS">
	<tls:context>
		<tls:key-store type="jks" path="server-keystore"
			keyPassword="mulepass" password="mulepass" />
	</tls:context>
</http:listener-config>
```

Note that the server connector requires one key-store which will contain the public self-signed certificate and private key of the service.

Next we define a client HTTPS request config:

```xml
<http:request-config protocol="HTTPS"
	name="httpsClientConnector" host="localhost" port="8083"
	doc:name="HTTP Request Configuration">
	<tls:context>
		<tls:trust-store path="client-truststore" password="mulepass"
			type="jks" />
	</tls:context>
</http:request-config>
```

This time we require to add a trust-store in the configuration which will include a key store containing the server certificate.

The Mule flows will now reference the above connectors:

```xml
<flow name="httpsClient">
	<http:listener path="httpServer" config-ref="httpServerConnector"
		doc:name="HTTP" />
	<logger message="sending request from https client to https server..."
		level="INFO" doc:name="Logger" />
	<http:request config-ref="httpsClientConnector" path="httpsServer"
		method="GET" doc:name="HTTP" />
</flow>
```

```xml
<flow name="httpsServer">
	<http:listener path="httpsServer" config-ref="httpsServerConnector"
		doc:name="HTTP" />
	<logger message="accessed https server successfully!" level="INFO"
		doc:name="Logger" />
</flow>
```