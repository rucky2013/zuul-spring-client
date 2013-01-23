[![Build Status](https://travis-ci.org/mcantrell/zuul-spring-client.png?branch=master)](https://travis-ci.org/mcantrell/zuul-spring-client)

# Zuul Spring Client

This project provides Spring helpers and namespaces for integrating with the web services provided by the
[Zuul Project](https://github.com/mcantrell/Zuul/wiki).


**Maven Dependency**
```xml
<groupId>org.devnull</groupId>
<artifactId>zuul-spring-client</artifactId>
<version>1.4</version>
```

<blockquote>
Starting with v 1.4 of the zuul-spring-client, the namespace has been refactored to allow for PGP and PBE key configuration.
The [older versions](https://github.com/mcantrell/zuul-spring-client/tree/1.3.x) will still work but do not support PGP.
</blockquote>

**context.xml**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:zuul="http://www.devnull.org/schema/zuul-spring-client"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
http://www.devnull.org/schema/zuul-spring-client http://www.devnull.org/schema/zuul-spring-client-1.3.xsd
http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.1.xsd">


    <context:property-placeholder properties-ref="appDataConfig"/>
    <zuul:properties id="appDataConfig" config="app-data-config" environment="prod">
        <zuul:file-store/>
        <zuul:pbe-decryptor password="secret" algorithm="PBEWITHSHA256AND128BITAES-CBC-BC"/>
        <!-- or use the pgp decryptor
           <zuul:pgp-decryptor password="#{environment['GNUPGPASSWD']}" secret-key-ring="#{environment['GNUPGHOME']}/secring.gpg"/>
        -->
    </zuul:properties>
</beans>
```

# Dynamic Configuration of Environment, etc.

**Spring Profiles**

Utilize [spring profiles](http://static.springsource.org/spring/docs/3.1.x/spring-framework-reference/htmlsingle/spring-framework-reference.html#testcontext-ctx-management-env-profiles)
to enable configuration by profile.

```xml
    <beans profile="prod">
        <context:property-placeholder properties-ref="appDataConfig"/>
        <zuul:properties id="appDataConfig" host="zuul.acme.com" config="test-aes-config" port="8081" environment="prod" ssl="false">
            <zuul:pbe-decryptor algorithm="PBEWITHSHA256AND128BITAES-CBC-BC" password="prodpassword"/>
        </zuul:properties>
    </beans>
    <beans profile="qa">
        <context:property-placeholder properties-ref="appDataConfig"/>
        <zuul:properties id="appDataConfig" host="zuul.acme.com" config="test-aes-config" port="8081" environment="qa" ssl="false"/>
    </beans>
    <beans profile="dev">
        <context:property-placeholder properties-ref="appDataConfig"/>
        <zuul:properties id="appDataConfig" host="zuul.acme.com" config="test-aes-config" port="8081" environment="dev" ssl="false"/>
    </beans
```

**Spring Expression Language**

Use environment variables to read in the password and environment:

```xml
    <context:property-placeholder properties-ref="appDataConfig"/>
    <zuul:properties id="appDataConfig" config="app-data-config" environment="#{environment['ZUUL_ENVIRONMENT']}">
        <zuul:file-store/>
        <zuul:pbe-decryptor password="#{environment['ZUUL_PASSWORD']}" algorithm="PBEWITHSHA256AND128BITAES-CBC-BC"/>
    </zuul:properties>
```

# Spring Namespace Reference


**zuul:properties attributes**
<table>
	<thead>
		<tr>
			<th>Attribute</th>
			<th>Description</th>
			<th>Default</th>
			<th>Required</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>config</td>
			<td>Name of the configuration to render</td>
			<td>n/a</td>
			<td>true</td>
		<tr>
		<tr>
			<td>host</td>
			<td>DNS or IP address of the zuul server</td>
			<td>localhost</td>
			<td>false</td>
		<tr>
		<tr>
			<td>port</td>
			<td>TCP port where the server is running</td>
			<td>80</td>
			<td>false</td>
		<tr>
		<tr>
			<td>context</td>
			<td>URI path to the root zuul application</td>
			<td>/zuul</td>
			<td>false</td>
		<tr>
		<tr>
			<td>environment</td>
			<td>Which environment set to retrieve</td>
			<td>dev</td>
			<td>false</td>
		<tr>
		<tr>
			<td>ssl</td>
			<td>Set to true if zuul endpoints are hosted via HTTPS</td>
			<td>false</td>
			<td>false</td>
		<tr>
		<tr>
			<td>http-client-ref</td>
			<td>Reference to a custom httpcomponents http-client</td>
			<td>A default client is created by default. You can override if needed</td>
			<td>false</td>
		<tr>
	</tbody>
</table>
<hr/>

**zuul:file-store attributes**

The zuul:file-store element is optional. It caches copies of the files (with encrypted values) to the local filesystem. If configured, it will be used as a backup strategy if the zuul web services are unavailable.

_If left un-configured, the application will throw an exception upon startup if the service is not available._

<table>
	<thead>
		<tr>
			<th>Attribute</th>
			<th>Description</th>
			<th>Default</th>
			<th>Required</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>path</td>
			<td>File Resource to contain the cached files.</td>
			<td>Uses the java.io.tmp system property by default</td>
			<td>false</td>
		<tr>
	</tbody>
</table>
<hr/>

**Decryption**

You have two options for decrypting values:


* zuul:pbe-decryptor

<blockquote>
Use this option if your configuration in Zuul has encrypted values from a PBE (password base encryption) key
such as AES, TripleDES, etc.
</blockquote>

<table>
	<thead>
		<tr>
			<th>Attribute</th>
			<th>Description</th>
			<th>Default</th>
			<th>Required</th>
		</tr>
	</thead>
	<tbody>
        <tr>
            <td>algorithm</td>
            <td>
                <p>
                    Provide an encryption algorithm which matches the Zuul key. Available values:
                </p>
                <ul>
                    <li>PBEWITHSHA256AND128BITAES-CBC-BC (AES Bouncy Castle)</li>
                    <li>PBEWithSHAAnd2-KeyTripleDES-CBC (Triple DES Bouncy Castle)</li>
                    <li>PBEWithMD5AndTripleDES (Triple DES JCE)</li>
                    <li>PBEWithMD5AndDES (DES JCE)</li>
                </ul>
                <p>
                    See the following for more information:
                </p>
                <ul>
                    <li><a href="http://www.jasypt.org/encrypting-configuration.html">Jasypt Documentation</a></li>
                    <li><a href="https://github.com/mcantrell/Zuul/wiki/Encryption">Zuul Encryption Documentation</a></li>
                <ul>
            </td>
            <td>null</td>
            <td>true</td>
        <tr>
        <tr>
            <td>password</td>
            <td>Shared, private password used to decrypt the values</td>
            <td>null</td>
            <td>true</td>
        <tr>
	</tbody>
</table>
<hr/>



* zuul:pgp-decryptor

<blockquote>
Use this option if your configuration in Zuul has encrypted values from a PGP key.
</blockquote>

<table>
	<thead>
		<tr>
			<th>Attribute</th>
			<th>Description</th>
			<th>Default</th>
			<th>Required</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>secretKeyRing</td>
			<td>File resource representing the PGP secret key ring (secring.gpg)</td>
			<td>null</td>
			<td>true</td>
		<tr>
        <tr>
            <td>password</td>
            <td>Password used to unlock the secret key ring (if encrypted)</td>
            <td>empty</td>
            <td>false</td>
        <tr>
	</tbody>
</table>


# License

   Copyright 2012 Mike Cantrell

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.
