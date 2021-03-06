<!--
  ~ Licensed to the Apache Software Foundation (ASF) under one
  ~ or more contributor license agreements.  See the NOTICE file
  ~ distributed with this work for additional information
  ~ regarding copyright ownership.  The ASF licenses this file
  ~ to you under the Apache License, Version 2.0 (the
  ~ "License"); you may not use this file except in compliance
  ~ with the License.  You may obtain a copy of the License at
  ~
  ~   http://www.apache.org/licenses/LICENSE-2.0
  ~
  ~ Unless required by applicable law or agreed to in writing,
  ~ software distributed under the License is distributed on an
  ~ "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
  ~ KIND, either express or implied.  See the License for the
  ~ specific language governing permissions and limitations
  ~ under the License.
  -->

---
layout: doc_page
---

TLS Support
===============

# General Configuration

|Property|Description|Default|
|--------|-----------|-------|
|`druid.enablePlaintextPort`|Enable/Disable HTTP connector.|`true`|
|`druid.enableTlsPort`|Enable/Disable HTTPS connector.|`false`|

Although not recommended but both HTTP and HTTPS connectors can be enabled at a time and respective ports are configurable using `druid.plaintextPort`
and `druid.tlsPort` properties on each node. Please see `Configuration` section of individual nodes to check the valid and default values for these ports.

# Jetty Server TLS Configuration

Druid uses Jetty as an embedded web server. To get familiar with TLS/SSL in general and related concepts like Certificates etc.
reading this [Jetty documentation](http://www.eclipse.org/jetty/documentation/9.3.x/configuring-ssl.html) might be helpful.
To get more in depth knowledge of TLS/SSL support in Java in general, please refer to this [guide](http://docs.oracle.com/javase/8/docs/technotes/guides/security/jsse/JSSERefGuide.html).
The documentation [here](http://www.eclipse.org/jetty/documentation/9.3.x/configuring-ssl.html#configuring-sslcontextfactory)
can help in understanding TLS/SSL configurations listed below. This [document](http://docs.oracle.com/javase/8/docs/technotes/guides/security/StandardNames.html) lists all the possible
values for the below mentioned configs among others provided by Java implementation.

|Property|Description|Default|Required|
|--------|-----------|-------|--------|
|`druid.server.https.keyStorePath`|The file path or URL of the TLS/SSL Key store.|none|yes|
|`druid.server.https.keyStoreType`|The type of the key store.|none|yes|
|`druid.server.https.certAlias`|Alias of TLS/SSL certificate for the connector.|none|yes|
|`druid.server.https.keyStorePassword`|The [Password Provider](../operations/password-provider.html) or String password for the Key Store.|none|yes|

The following table contains configuration options related to client certificate authentication.

|Property|Description|Default|Required|
|--------|-----------|-------|--------|
|`druid.server.https.requireClientCertificate`|If set to true, clients must identify themselves by providing a TLS certificate.  If `requireClientCertificate` is false, the rest of the options in this table are ignored.|false|no|
|`druid.server.https.trustStoreType`|The type of the trust store containing certificates used to validate client certificates. Not needed if `requireClientCertificate` is false.|`java.security.KeyStore.getDefaultType()`|no|
|`druid.server.https.trustStorePath`|The file path or URL of the trust store containing certificates used to validate client certificates. Not needed if `requireClientCertificate` is false.|none|yes, only if `requireClientCertificate` is true|
|`druid.server.https.trustStoreAlgorithm`|Algorithm to be used by TrustManager to validate client certificate chains. Not needed if `requireClientCertificate` is false.|`javax.net.ssl.TrustManagerFactory.getDefaultAlgorithm()`|no|
|`druid.server.https.trustStorePassword`|The [Password Provider](../../operations/password-provider.html) or String password for the Trust Store.  Not needed if `requireClientCertificate` is false.|none|no|
|`druid.server.https.validateHostnames`|If set to true, check that the client's hostname matches the CN/subjectAltNames in the client certificate.  Not used if `requireClientCertificate` is false.|true|no|
|`druid.server.https.crlPath`|Specifies a path to a file containing static [Certificate Revocation Lists](https://en.wikipedia.org/wiki/Certificate_revocation_list), used to check if a client certificate has been revoked. Not used if `requireClientCertificate` is false.|null|no|

The following table contains non-mandatory advanced configuration options, use caution.

|Property|Description|Default|Required|
|--------|-----------|-------|--------|
|`druid.server.https.keyManagerFactoryAlgorithm`|Algorithm to use for creating KeyManager, more details [here](https://docs.oracle.com/javase/7/docs/technotes/guides/security/jsse/JSSERefGuide.html#KeyManager).|`javax.net.ssl.KeyManagerFactory.getDefaultAlgorithm()`|no|
|`druid.server.https.keyManagerPassword`|The [Password Provider](../operations/password-provider.html) or String password for the Key Manager.|none|no|
|`druid.server.https.includeCipherSuites`|List of cipher suite names to include. You can either use the exact cipher suite name or a regular expression.|Jetty's default include cipher list|no|
|`druid.server.https.excludeCipherSuites`|List of cipher suite names to exclude. You can either use the exact cipher suite name or a regular expression.|Jetty's default exclude cipher list|no|
|`druid.server.https.includeProtocols`|List of exact protocols names to include.|Jetty's default include protocol list|no|
|`druid.server.https.excludeProtocols`|List of exact protocols names to exclude.|Jetty's default exclude protocol list|no|

# Druid's internal communication over TLS

Whenever possible Druid nodes will use HTTPS to talk to each other. To enable this communication Druid's HttpClient needs to
be configured with a proper [SSLContext](http://docs.oracle.com/javase/8/docs/api/javax/net/ssl/SSLContext.html) that is able
to validate the Server Certificates, otherwise communication will fail.

Since, there are various ways to configure SSLContext, by default, Druid looks for an instance of SSLContext Guice binding
while creating the HttpClient. This binding can be achieved writing a [Druid extension](../development/extensions.html)
which can provide an instance of SSLContext. Druid comes with a simple extension present [here](../development/extensions-core/simple-client-sslcontext.html)
which should be useful enough for most simple cases, see [this](./including-extensions.html) for how to include extensions.
If this extension does not satisfy the requirements then please follow the extension [implementation](https://github.com/apache/incubator-druid/tree/master/extensions-core/simple-client-sslcontext)
to create your own extension.

# Upgrading Clients that interact with Overlord or Coordinator
When Druid Coordinator/Overlord have both HTTP and HTTPS enabled and Client sends request to non-leader node, then Client is always redirected to the HTTPS endpoint on leader node.
So, Clients should be first upgraded to be able to handle redirect to HTTPS. Then Druid Overlord/Coordinator should be upgraded and configured to run both HTTP and HTTPS ports. Then Client configuration should be changed to refer to Druid Coordinator/Overlord via the HTTPS endpoint and then HTTP port on Druid Coordinator/Overlord should be disabled.

