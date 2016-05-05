## Install AEM
```
java -jar cq-author-4502.jar`
    Loading quickstart properties: default
    Loading quickstart properties: instance
    Low-memory action set to fork
    Using 64bit VM settings, min.heap=1024MB, min permgen=256MB, default fork arguments=[-Xmx1024m, -XX:MaxPermSize=256m]
    The JVM reports a heap size of 3641 MB, meets our expectation of 1024 MB +/- 20
    Setting properties from filename '/Users/cpilsworth/dev/apps/aem-coldstandby/primary/cq-author-4502.jar'
    Option '-quickstart.server.port' set to '4502' from filename cq-author-4502.jar
    Verbose option not active, closing stdin and redirecting stdout and stderr
    Redirecting stdout to /Users/cpilsworth/dev/apps/aem-coldstandby/primary/crx-quickstart/logs/stdout.log
    Redirecting stderr to /Users/cpilsworth/dev/apps/aem-coldstandby/primary/crx-quickstart/logs/stderr.log
    Press CTRL-C to shutdown the Quickstart server...
```

## Check it is up and running
```
curl -uadmin:admin -I http://localhost:4502/projects.html`
    HTTP/1.1 200 OK
    Date: Thu, 05 May 2016 13:00:15 GMT
    X-Content-Type-Options: nosniff
    Set-Cookie: cq-authoring-mode=TOUCH;Path=/;Expires=Thu, 12-May-2016 13:00:15 GMT
    Expires: Thu, 01 Jan 1970 00:00:00 GMT
    Content-Type: text/html; charset=UTF-8
    X-UA-Compatible: IE=edge
    Transfer-Encoding: chunked
    Server: Jetty(9.2.9.v20150224)
    X-Powered-By: Jetty(9.2.9.v20150224)
```

## Ctrl+C for AEM instance to shutdown

## Copy the primary config and start the primary
```
cp -r config/install.primary/ primary/crx-quickstart/install/install.primary

CQ_RUNMODE=primary primary/crx-quickstart/bin/start
```

## Copy standby config and start standby server
```
cp -r config/install.standby/ primary/crx-quickstart/install/install.standby

CQ_PORT=4504 CQ_RUNMODE=standby standby/crx-quickstart/bin/start
```

## Check the processes are running with the correct runmode/port
```
ps  -ef |grep java | grep -v grep

  501 89829     1   0  2:45pm ttys006    1:05.04 /usr/bin/java -server -Xmx1024m -XX:MaxPermSize=256M -Djava.awt.headless=true -Dsling.run.modes=standby,crx3,crx3tar -jar crx-quickstart/app/cq-quickstart-6.1.0-standalone-quickstart.jar start -c crx-quickstart -i launchpad -p 4504 -Dsling.properties=conf/sling.properties
  501 89991     1   0  2:46pm ttys006    1:15.98 /usr/bin/java -server -Xmx1024m -XX:MaxPermSize=256M -Djava.awt.headless=true -Dsling.run.modes=primary,crx3,crx3tar -jar crx-quickstart/app/cq-quickstart-6.1.0-standalone-quickstart.jar start -c crx-quickstart -i launchpad -p 4502 -Dsling.properties=conf/sling.properties
```  

## Check Standby status
`open http://localhost:4504/system/console/jmx/org.apache.jackrabbit.oak%3Aid%3D%223981d89a-1e2e-4e3d-adb2-f2673ff8aab5%22%2Cname%3DStatus%2Ctype%3D%22Standby%22`

## Check Primary status
`open http://localhost:4502/system/console/jmx/org.apache.jackrabbit.oak%3Aid%3D8023%2Cname%3DStatus%2Ctype%3D%22Standby%22`
Note: Check the status of the client by searching for "standby" on the jmx system console.  Clients get assigned a unique guid.

## Create some pages, edit some pages, etc
Login to the author instance, create/copy some pages

## Stop the instances
```
primary/crx-quickstart/bin/stop
  05.05.2016 15:00:41.038 *INFO * [main] Setting sling.home=. (command line)
  05.05.2016 15:00:41.054 *INFO * [main] Sent 'stop' to /127.0.0.1:58558: OK
  Application not running

standby/crx-quickstart/bin/stop
  05.05.2016 15:00:48.560 *INFO * [main] Setting sling.home=. (command line)
  05.05.2016 15:00:48.588 *INFO * [main] Sent 'stop' to /127.0.0.1:58244: OK
  Application not running
```

## Create a copy of the standby
`cp -r standby newstandby`

## Copy the config to the new standby
`cp -r config/install.standby/ newstandby/crx-quickstart/install/install.standby`

## Copy the primary config to the old standby & remove standby config
```
cp -r config/install.primary/ standby/crx-quickstart/install/install.primary
rm -rf standby/crx-quickstart/install/install.standby/
```

## Start the standby as the new primary
`CQ_RUNMODE=primary standby/crx-quickstart/bin/start`

## Start the newstandby as the new standby
`CQ_PORT=4504 CQ_RUNMODE=standby newstandby/crx-quickstart/bin/start`

## Verify CMS access on the new primary
```
curl -I -uadmin:admin http://localhost:4502/projects.html
HTTP/1.1 200 OK
Date: Thu, 05 May 2016 14:53:30 GMT
X-Content-Type-Options: nosniff
Set-Cookie: cq-authoring-mode=TOUCH;Path=/;Expires=Thu, 12-May-2016 14:53:30 GMT
Expires: Thu, 01 Jan 1970 00:00:00 GMT
Content-Type: text/html; charset=UTF-8
X-UA-Compatible: IE=edge
Transfer-Encoding: chunked
Server: Jetty(9.2.9.v20150224)
X-Powered-By: Jetty(9.2.9.v20150224)
```

## Verify console access on the new standby
```
curl -I -uadmin:admin http://localhost:4504/system/console/jmx
HTTP/1.1 200 OK
Date: Thu, 05 May 2016 14:54:04 GMT
Set-Cookie: felix-webconsole-locale=en;Path=/system/console;Expires=Wed, 30-Apr-2036 14:54:04 GMT
Expires: Thu, 01 Jan 1970 00:00:00 GMT
Content-Type: text/html; charset=UTF-8
Content-Length: 40174
Server: Jetty(9.2.9.v20150224)
X-Powered-By: Jetty(9.2.9.v20150224)
```

*The final directory structure should look something like this (abridged):*
```
├── README.md
├── config
│   ├── install.primary
│   │   ├── org.apache.jackrabbit.oak.plugins.segment.SegmentNodeStoreService.config
│   │   └── org.apache.jackrabbit.oak.plugins.segment.standby.store.StandbyStoreService.config
│   └── install.standby
│       ├── org.apache.jackrabbit.oak.plugins.segment.SegmentNodeStoreService.config
│       └── org.apache.jackrabbit.oak.plugins.segment.standby.store.StandbyStoreService.config
├── primary
│   ├── cq-author-4502.jar
│   ├── crx-quickstart
│   │   ├── install
│   │   │   ├── install.primary
│   │   └── repository
│   │       ├── index
│   │       └── segmentstore
│   └── license.properties
└── standby
│   ├── cq-author-4502.jar
│   ├── crx-quickstart
│   │   ├── install
│   │   │   └── install.primary
│   │   └── repository
│   │       ├── index
│   │       └── segmentstore
│   └── license.properties
├── newstandby
│   ├── cq-author-4502.jar
│   ├── crx-quickstart
│   │   │   └── install.standby
│   │   └── repository
│   │       ├── index
│   │       └── segmentstore
│   └── license.properties    
```
