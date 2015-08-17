# mule-https-example-3.7.x

This page describes how to define an HTTPS client and server flow in Mule 3.7. I explain both how to configure one-way SSL as well as mutual authentication by giving both client and server example flows in Mule.

Note, all key stores and trust stores in this example have been generated using keystore explorer which is a very handy graphical alternative to the java keytool. All stores were created using the RSA algorithm.

The following is an HTTP listener used to accept messages in a client flow:

```xml
<http:listener-config name="httpServerConnector"
	host="localhost" port="8081" doc:name="HTTP Listener Configuration" />
```

**One-way SSL**

The following snippet is an HTTPS server listener configured for one way SSL. The key store contains the public certificate and private key.

```xml
<http:listener-config name="httpsServerConnectorOneWaySSL"
	host="localhost" port="8082" doc:name="HTTP Listener Configuration"
	protocol="HTTPS">
	<tls:context>
		<tls:key-store type="jks" path="server-keystore"
			keyPassword="mulepass" password="mulepass" />
	</tls:context>
</http:listener-config>
```
The following is an HTTPS request configuration for one-way SSL. The trust store contains the public certificate of the HTTPS server flow above.

```xml
<http:request-config protocol="HTTPS"
	name="httpsClientConnectorOneWaySSL" host="localhost" port="8082"
	doc:name="HTTP Request Configuration">
	<tls:context>
		<tls:trust-store path="client-truststore" password="mulepass"
			type="jks" />
	</tls:context>
</http:request-config>
```

**Mutual Authentication**

For mutual authentication we need to configure both client and server connectors using both a key store as well as trust store. Lets take a look at what each of these shall contain.

The HTTPS server flow again holds a reference to the keystore which stores the public selfs signed certificate and private key. In addition, we reference the server trust store which stores the certifictes of client applications which it should trust.

```xml
<http:listener-config name="httpsServerConnectorMutualAuth"
	host="localhost" port="8083" doc:name="HTTP Listener Configuration"
	protocol="HTTPS">
	<tls:context>
		<tls:trust-store type="jks" path="server-truststore"
			password="mulepass" />
		<tls:key-store type="jks" path="server-keystore"
			keyPassword="mulepass" password="mulepass" />
	</tls:context>
</http:listener-config>
```

The following snippet is the HTTPS request config referencing the client trust store (same as when we set up one-was SSL this holds the certificate of the HTTPS server). This time we also specify a client keystore which includes a public certificate and private key which belong to the client. 

```xml
<http:request-config protocol="HTTPS"
	name="httpsClientConnectorMutualAuth" host="localhost" port="8083"
	doc:name="HTTP Request Configuration">
	<tls:context>
		<tls:trust-store path="client-truststore" password="mulepass"
			type="jks" />
		<tls:key-store type="jks" path="client-keystore"
			keyPassword="mulepass" password="mulepass" />
	</tls:context>
</http:request-config>
```
	
Below are Flow samples of how to use the above connectors to implement one-way SSL and mutual authentication. To test the flows simply send an HTTPS GET request to the URLs: http://localhost:8081/oneWaySSL or http://localhost:8081/mutualAuth.

**One way SSL flows:**

```xml
<flow name="httpsClientOneWaySSL">
	<http:listener path="oneWaySSL" config-ref="httpServerConnector"
		doc:name="HTTP" />
	<logger message="sending request from https client to https server (one way ssl)..."
		level="INFO" doc:name="Logger" />
	<http:request config-ref="httpsClientConnectorOneWaySSL"
		path="httpsServer" method="GET" doc:name="HTTP" />
</flow>
```

```xml
<flow name="httpsServerOneWaySSL">
	<http:listener path="httpsServer" config-ref="httpsServerConnectorOneWaySSL"
		doc:name="HTTP" />
	<logger message="accessed https server successfully - via one way ssl!" level="INFO"
		doc:name="Logger" />
</flow>
```

**Mutual Auth Flows:**

```xml
<flow name="httpsClientMutualAuth">
	<http:listener path="mutualAuth" config-ref="httpServerConnector"
		doc:name="HTTP" />
	<logger message="sending request from https client to https server (mutual auth)..."
		level="INFO" doc:name="Logger" />
	<http:request config-ref="httpsClientConnectorMutualAuth"
		path="httpsServer" method="GET" doc:name="HTTP" />
</flow>

```xml
<flow name="httpsServerMutualAuth">
	<http:listener path="httpsServer" config-ref="httpsServerConnectorMutualAuth"
		doc:name="HTTP" />
	<logger message="accessed https server successfully - via mutual authentication!" level="INFO"
		doc:name="Logger" />
</flow>
```
