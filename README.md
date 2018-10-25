# Gemfire demo with 2 way SSL authentication

## Create required keystore and truststore using java utility - keytool.

### Create server keystore named MyServer.jks.
keytool -genkey -alias MyServer -keyalg RSA -validity 1825 -keystore "MyServer.jks" -storetype JKS -dname "CN=myserver.com,OU=My Company Name,O=My Organization,L=My Location,ST=My State,C=My Country Short Code" -keypass password -storepass password

### Export server's public certificate. This will be kept in client's truststore for client to authC server.
keytool -exportcert -alias MyServer -keystore MyServer.jks -file MyServer.cer

### Create client keystore named MyClient.jks
keytool -genkey -alias MyClient -keyalg RSA -validity 1825 -keystore MyClient.jks -storetype JKS -dname "CN=client.com,OU=Client Company,O=Client,L=CLient Location,ST=Client State,C=Client Country Short Code" -keypass password -storepass password

### Export client's public certificate. This will be kept in server's truststore for server to authC client.
keytool -exportcert -alias MyClient -keystore MyClient.jks -file MyClientPublic.cer

### Add Server certificate to client truststore
keytool -importcert -alias MyServer -keystore MyClient.jks -file MyServer.cer

### Add client certificate to server truststore
keytool -importcert -alias MyClient -keystore MyServer.jks -file MyClientPublic.cer
## GemFire secure (SSL enabled) cluster setup.

### References:
#### 0. https://gemfire.docs.pivotal.io/96/geode/managing/security/ssl_example.html
#### 1. http://www.ossmentor.com/2015/03/one-way-and-two-way-ssl-and-tls.html
#### 2. https://community.pivotal.io/s/article/How-to-use-a-simple-method-to-create-a-self-sign-SSL-Keystore-and-Truststore
#### 3. https://github.com/Pivotal-Field-Engineering/pad-gemfire-ssl-client/blob/master/src/main/java/io/pivotal/gemfire/demo/GemfireSslClientApplication.java
#### 4. https://www.opencodez.com/java/implement-2-way-authentication-using-ssl.htm
#### 5. https://pubs.vmware.com/continuent/tungsten-replicator-3.0/deployment-ssl-stores.html
#### 6. https://kb.informatica.com/h2l/HowTo%20Library/1/0312_SSL_Two-Way_Configuration.pdf

## Create secured (SSL enabled) gemfire cluster

### gemfire.properties
```
ssl-enabled-components=all
mcast-port=0
locators=localhost[10334]
```

### gfsecurity.properties
```
ssl-keystore=/home/nilkanth/work/pivotal-gemfire-9.3.0/myjks/MyServer.jks
ssl-keystore-password=password
ssl-truststore=/home/nilkanth/work/pivotal-gemfire-9.3.0/myjks/MyServer.jks
ssl-truststore-password=password
##security-username=xxxx
##security-userPassword=yyyy
```

### Steps to start secure cluster

```
//start locator
start locator --name=mylocator --properties-file=/home/nilkanth/work/pivotal-gemfire-9.3.0/mutualssl/gemfire.properties --security-properties-file=/home/nilkanth/work/pivotal-gemfire-9.3.0/mutualssl/gfsecurity.properties

//start cache-server
start server --name=myserver --properties-file=/home/nilkanth/work/pivotal-gemfire-9.3.0/mutualssl/gemfire.properties --security-properties-file=/home/nilkanth/work/pivotal-gemfire-9.3.0/mutualssl/gfsecurity.properties


//connecting to ssl secured cluster from gfsh
connect --locator=localhost[10334] --use-ssl --security-properties-file=/path/to/your/gfsecurity.properties

OR

gfsh>connect --locator=localhost[10334] --use-ssl
key-store: /home/nilkanth/work/pivotal-gemfire-9.3.0/myjks/MyClient.jks
key-store-password: ********
key-store-type(default: JKS):
trust-store: /home/nilkanth/work/pivotal-gemfire-9.3.0/myjks/MyClient.jks
trust-store-password: ********
trust-store-type(default: JKS):
ssl-ciphers(default: any):
ssl-protocols(default: any):
ssl-enabled-components(default: all):
Connecting to Locator at [host=localhost, port=10334] ..
Connecting to Manager at [host=172.16.160.102, port=1099] ..
Successfully connected to: [host=172.16.160.102, port=1099]
```
