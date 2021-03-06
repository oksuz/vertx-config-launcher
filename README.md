# Vert.x Config Launcher

This project aims to provide easy way to launch vert.x instances. It just reads config file as Freemarker Template and 
renders it with JVM System Properties. 

Also please note that we think defining default behaviour for application may result 
inconsistencies at production environment, that is why we do not read default-cluster.xml file by default
If application is going to be launched as clustered then cluster.xml file must be provided explicitly 
by defining -Dcluster-xml jvm argument.

### Dependencies

Add necessary such as dependencies
 
```groovy
    compile "io.vertx:vertx-core:3.3.2"
	compile 'io.vertx:vertx-rx-java:3.3.2'
	compile 'org.freemarker:freemarker:2.3.23'
	runtime 'org.slf4j:slf4j-log4j12:1.7.5'
    compile 'org.slf4j:jcl-over-slf4j:1.7.5'
    compile 'org.apache.logging.log4j:log4j:2.5'

    //for clustered usage add these besides necessary ones
	compile 'org.apache.curator:curator-x-discovery:2.11.0'
	compile 'org.apache.curator:curator-test:2.11.0'
	compile "io.vertx:vertx-hazelcast:3.3.2"
	compile 'com.hazelcast:hazelcast-zookeeper:3.6.1'
	compile 'com.hazelcast:hazelcast-client:3.6.3'
```

### build.gradle Config

Set your application's mainClass as launcher's. And define your jvm default arguements, those will be used for rendering 
config file. For example if config file includes ${mongoConfig} then when you pass -DmongoOptions=${file('conf/mongoConf.json').text}
content of conf/mongoConf.json will be rendered. For what else can be done using template language please refer to [Freemarker]
template language documentation.


```groovy
plugins {
	id 'application'
	
}


mainClassName = 'com.foreks.vertx.launcher.VertxConfigLauncher'

/* !!!OPTIONAL!!!
 Place your default jvm args, 
 you can pass JVM Property and use it with your config file 
 check out below clusterHost field for how we use it
*/
applicationDefaultJvmArgs = [
    "-DmongoOptions=${file('conf/mongoConf.json').text}", // we can even render config reading file contents
	'-Dnodeip=127.0.0.1',
	'-Dcluster-xml=conf/cluster.xml' // if Vert.x is clustered,
	                                 // Than cluster-xml JVM arg must be provided
	                                 
	'-Dconf=conf/config.json',       // applicaton config file checkout below
	'-Dlog4j.configuration=file:conf/log4j.xml',
	...
	
	
	]

```

Place this dependency inside your build.gradle

```groovy

repositories {
	mavenCentral()
}

dependencies {
    
    compile 'com.foreks:vertx-config-launcher:0.9.0'
    
}

```

### conf/config.json

Config file should include [VertxOptions] and Verticle's [DeploymentOptions] in following way

> ####deploymentOptions

>config -> Verticle spesific config goes under deploymentOptions/config
this will be reached as JsonObject by Verticle 
when config() called inside the Verticle

>instances -> spesify how many instance will be deployed

>ha -> set true if verticle intented to be High Available

>worker -> If this verticle is doing long running tasks
                                    then this should be true, so that we let WorkerEventPool
                                    to handle these tasks
                                    
>multiThreaded -> // If worker is true then Verticle may be called from different threads. 
                                              Don't set this true if you don't know what you are doing
                                              in ideal situation each Verticle should be called by only one thread
					

>Please refer to documentation for other options 
which are extraClasspath, isolatedClasses, isolationGroup, maxWorkerExecuteTime
multiThreaded, worker, workerPoolName, workerPoolSize
 
> ####vertxOptions

>clustered -> If this field is true than vertx will run clustered so cluster.xml must provide

>clusterHost -> cluster host name, (OPTIONAL you can pass things like this"${nodeip}")

>quorumSize -> Untill quorum size satisfied verticle is not gonna be deployed

>haEnabled -> Set true if Vert.x should be High Available

>definition -> If vertx is ha than this can be grouped under this key

>eventLoopPoolSize -> Main Event Loop pool size, should be equal to CPU core size

>workerPoolSize -> If Vert.x has a lot of long running tasks
                                Then Worker Event Pool should handle those
                                And this can be greater than CPU core size
                                
>Other options are addressResolverOptions, blockedThreadCheckInterval, clusterPingInterval, clusterPort
clusterPublicHost, clusterPublicPort, eventBusOptions, internalBlockingPoolSize,
maxEventLoopExecuteTime, maxWorkerExecuteTime, metricsOptions, warningExceptionTime              

```json
{
	"verticles": {
		"com.foreks.feed.tip.filereader.FirstVerticle": {
			"deploymentOptions": {
				"config": {
					"mongoClient":${mongoOptions}
				}
				},
				"instances": 1, 
				"ha": true,
				"worker": false, 
				"multiThreaded": false 
		},
		"com.foreks.feed.tip.filereader.SecondVerticle": {
        			"deploymentOptions": {
        				"config": {
        				    "mongoClient":${mongoOptions},
        					"fileOpenOptions":{
        						"read":true,
        						"write":false
        					},
        					"filePath":"data/input.log",
        					"address":"tip-file-reader"
        				}
        				},
        				"instances": 1,
        				"ha": true,
        				"worker": false,
        				"multiThreaded": false
        		}
	},
	"vertxOptions": {
		"clustered": true, 
		"clusterHost": "${nodeip}",
		"quorumSize": 1,
		"haEnabled": true, 
		"haGroup": "definition",
		"eventLoopPoolSize": 4,
		"workerPoolSize": 12,
	}
}

```

### Todos

 - Etcd integration, instead Jvm default arguements config can be rendered using etcd key value store

License
----

Copyright 2016 Sercan Karaoglu/Foreks

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.



   [DeploymentOptions]: <http://vertx.io/docs/apidocs/io/vertx/core/DeploymentOptions.html>
   [VertxOptions]: <http://vertx.io/docs/apidocs/io/vertx/core/VertxOptions.html>
   [Freemarker]: <http://freemarker.org/>