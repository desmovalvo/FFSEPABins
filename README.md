# FFSEPABins
The repository includes all you need to test FFSEPA, a publish-subscribe architecture designed to support information level interoperability by means of Semantic Web technologies. The architecture is built on top of a generic SPARQL endpoint where publishers and subscribers use standard **SPARQL** [Updates](https://www.w3.org/TR/sparql11-update/) and [Queries](https://www.w3.org/TR/sparql11-query/). Notifications about events (i.e., changes in the [**RDF**](https://www.w3.org/RDF/) knowledge base) are expressed in terms of [added and removed SPARQL binding results](http://wot.arces.unibo.it/TR/sparql11-subscribe.html) since the previous notification. This is a fork of SEPA.

Developers can benefit of a set of API implementing a [Producer-Aggregator-Consumer design pattern](http://wot.arces.unibo.it/TR/jsap.html).

For more details on the current implementation and how you can contribute please follow this [link](https://github.com/arces-wot/SEPA).

## Installation
Clone the repository: `git clone https://github.com/desmovalvo/FFSEPABins.git`

## Usage
The SEPA engine needs to connect to a SPARQL endpoint supporting the [SPARQL 1.1 Protocol](https://www.w3.org/TR/sparql11-protocol/). For you convenience, the repository includes two endpoints ([Blazegraph](https://www.blazegraph.com/),[Fuseki](https://jena.apache.org/documentation/serving_data/)).

### Start a SPARQL endpoint
If you want to use [Fuseki](https://jena.apache.org/documentation/serving_data/):
1. Move to the `fuseki` folder
2. Run the endpoint: `./fuseki-server --update --mem /sepa`

If you want to use [Blazegraph](https://www.blazegraph.com/):
1. Move to the `blazegraph` folder
2. Run the endpoint: `java -server -Xmx4g -jar blazegraph.jar`

### Start the SEPA engine
1. Move to the `engine` folder
2. The SEPA engine uses to configuration file: `endpoint.jar` and `engine.jpar`. The former is used to specify the parameters to interact with the SPARQL endpoint, the latter to set-up the engine parameters (e.g., ports, paths, timeouts, ...). In the Engine folder you find two configuration files for the two endpoints: `endpoint-fuseki.jpar` and `endpoint-blazegraph.jpar`. Rename to `endpoint.jpar` the one corresponding to the endpoint you choose to use.
3. Run the engine: `java -XX:+UseG1GC -Dlog4j.configurationFile=./log4j2.xml -jar SEPAEngine_X_Y_Z.jar`

<a name="running"></a>
```
##########################################################################################
# SEPA Engine Ver 0.8.3  Copyright (C) 2016-2017                                         #
# Web of Things Research @ ARCES - University of Bologna (Italy)                         #
#                                                                                        #
# This program comes with ABSOLUTELY NO WARRANTY                                         #
# This is free software, and you are welcome to redistribute it under certain conditions #
# GNU GENERAL PUBLIC LICENSE, Version 3, 29 June 2007                                    #
#                                                                                        #
# GITHUB: https://github.com/arces-wot/sepa                                              #
# WEB: http://wot.arces.unibo.it                                                         #
# WIKI: https: // github.com/arces-wot/SEPA/wiki                                         #
##########################################################################################

SPARQL 1.1 Protocol (https://www.w3.org/TR/sparql11-protocol/)
----------------------
SPARQL 1.1 Query     | http://192.168.1.7:8000/query
SPARQL 1.1 Update    | http://192.168.1.7:8000/update
----------------------

SPARQL 1.1 SE Protocol (https://wot.arces.unibo.it/TR/sparql11-se-protocol/)
----------------------
SPARQL 1.1 SE Query  | https://192.168.1.7:8443/secure/query
SPARQL 1.1 SE Update | https://192.168.1.7:8443/secure/update
Client registration  | https://192.168.1.7:8443/oauth/register
Token request        | https://192.168.1.7:8443/oauth/token
Subscribe            | ws://192.168.1.7:9000/subscribe
Secure Subscribe     | wss://192.168.1.7:9443/secure/subscribe
----------------------

*****************************************************************************************
*                      SEPA Engine Ver 0.8.3 is up and running                          *
*                                Let Things Talk!                                       *
*****************************************************************************************
```
## Configure
SEPA engine configuration parameters are stored in a JSON file (named `engine.jpar`) like the following:
```json
{
	"parameters" : {
		"scheduler" : {
			"queueSize" : 100}
		 ,
		"processor" : {
			"updateTimeout" : 5000 ,
			"queryTimeout" : 5000,
			"maxConcurrentRequests" : 5}
		 ,
		"spu" : {
			"keepalive" : 5000}
		 ,
		"ports" : {
			"http" : 8000 ,
			"ws" : 9000 ,
			"https" : 8443 ,
			"wss" : 9443}
		 ,
		"paths" : {
			"update" : "/update" ,
			"query" : "/query" ,
			"subscribe" : "/subscribe" ,
			"register" : "/oauth/register" ,
			"tokenRequest" : "/oauth/token" ,
			"securePath" : "/secure"}
	}
}
```
The `ports` and `paths` members are used to specify the URLs at which the engine is listening for requests. The above default configuration initializes the engine has shown [here](#running). The `keepalive` member specifies the ping period. Timeouts on update and query processing on the SPARQL endpoint are specified respectively by the `updateTimeout` and `queryTimeout` members. It also possible to setup the maximum number of concurrent requests that can be processed by the endpoint (see `maxConcurrentRequests`). Eventually, the `queueSize`is the maximum number of pending requests after which the engine starts to deny new requests.

## Security issues
The engine uses a JKS for storing the keys and certificates for [SSL](http://docs.oracle.com/cd/E19509-01/820-3503/6nf1il6ek/index.html) and [JWT](https://tools.ietf.org/html/rfc7519) signing/verification. A default `sepa.jks` is provided including a single X.509 certificate (the password for both the store and the key is: `sepa2017`). If you face problems using the provided JKS, please delete the `sepa.jks` file and create a new one as follows: `keytool -genkey -keyalg RSA -alias sepakey -keystore sepa.jks -storepass sepa2017 -validity 360 -keysize 2048`
The SEPA engine allows to use a user generated JKS. Run `java -jar engine-x.y.z.jar -help` for a list of options. The Java [Keytool](https://docs.oracle.com/javase/6/docs/technotes/tools/solaris/keytool.html) can be used to create, access and modify a JKS. 

## JMX monitoring
The SEPA engine is also distributed with a default [JMX](http://www.oracle.com/technetwork/articles/java/javamanagement-140525.html) configuration `jmx.properties` (including the `jmxremote.password` and `jmxremote.access` files for password and user grants). Remember to change password file permissions using: `chmod 600 jmxremote.password`. To enable remote JMX, the engine must be run as follows: `java -Dcom.sun.management.config.file=jmx.properties -jar engine-x.y.z.jar`. Using [`jconsole`](http://docs.oracle.com/javase/7/docs/technotes/guides/management/jconsole.html) is possible to monitor and control the most important engine parameters. By default, the port is `5555` and the `root:root` credentials grant full control (read/write).

## Experiment with the Dashboard
In the `Tools` folder, you can find an application (`SEPADashboard_X_Y_Z.jar`) that allows you to interact and experiment the functionalities offered by SEPA. Just double click on the jar or run it from a command shell as: `java -jar Dashboard_X_Y_Z.jar` 

SEPA applications are built around a [JSON Semantic Application Profile (JSAP)](http://wot.arces.unibo.it/TR/jsap.html). Examples of JSAP files can be found in the `jsap` folder. You should start by loading the `chat.jsap` file into the Dashboard...enjoy!
