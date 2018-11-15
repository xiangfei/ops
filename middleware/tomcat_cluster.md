<?xml version='1.0' encoding='utf-8'?>
<Server port="${shutdown.port}" shutdown="PAHZS8fu">
  <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
  <Listener className="org.apache.catalina.core.JasperListener" />
  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
  <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />
  
  <GlobalNamingResources>
    <Resource name="UserDatabase" auth="Container"
              type="org.apache.catalina.UserDatabase"
              description="User database that can be updated and saved"
              factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
              pathname="conf/tomcat-users.xml" />
  </GlobalNamingResources>

  <Service name="Catalina">
    <Executor maxThreads="1024"
               minSpareThreads="50"
               name="tomcatThreadPool"
               namePrefix="tomcat-threads--"/>

  <Connector acceptCount="800"
            port="${http.port}"
            protocol="HTTP/1.1"
            executor="tomcatThreadPool"
            enableLookups="false"
            connectionTimeout="20000"
            maxThreads="1024"
            disableUploadTimeout="true"
            URIEncoding="UTF-8"/>
  <Connector acceptCount="800"
            port="${ajp.port}"
            protocol="AJP/1.3"
            enableLookups="false"
            connectionTimeout="20000"
            maxThreads="1024"
            disableUploadTimeout="true"
            URIEncoding="UTF-8"
            useBodyEncodingForURI="true"/>

    <Engine name="Catalina" defaultHost="localhost" jvmRoute="${jvmRouteName}">
      <Realm className="org.apache.catalina.realm.LockOutRealm">
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm" resourceName="UserDatabase"/>
      </Realm>

      <Host name="localhost"  appBase="webapps" unpackWARs="true" autoDeploy="false">

      <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
             prefix="access." suffix=".log" pattern="%h %l %u %t %r %s %b %D" resolveHosts="false"/>
          
      <Context path="/${app.context}" docBase="${app.path.prefix}/${app.context}/${app.war.name}" reloadable="false"/>

      </Host>
    </Engine>
  </Service>
</Server>