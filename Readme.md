# dcm4che-weasis-integration

### _compose_

```yml
---
version: "3"
services:
  ldap:
    image: dcm4che/slapd-dcm4chee:2.6.5-31.2
    container_name: ldap
    logging:
      driver: json-file
      options:
        max-size: "10m"
    ports:
      - "389:389"
    environment:
      STORAGE_DIR: /storage/fs1
    volumes:
      - /var/local/dcm4chee-arc/ldap:/var/lib/openldap/openldap-data
      - /var/local/dcm4chee-arc/slapd.d:/etc/openldap/slapd.d
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 1024M
        reservations:
          cpus: '1'
          memory: 512M    
    networks:
      - pacs-network
  db:
    image: dcm4che/postgres-dcm4chee:15.4-31
    container_name: db
    logging:
      driver: json-file
      options:
        max-size: "10m"
    environment:
      POSTGRES_DB: pacsdb
      POSTGRES_USER: pacs
      POSTGRES_PASSWORD: pacs
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
      - /var/local/dcm4chee-arc/db:/var/lib/postgresql/data
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 1024M
        reservations:
          cpus: '1'
          memory: 512M    
    networks:
      - pacs-network
  arc:
    image: dcm4che/dcm4chee-arc-psql:5.31.2
    container_name: arc
    logging:
      driver: json-file
      options:
        max-size: "10m"
    ports:
      - "8080:8080"
      - "8443:8443"
      - "9990:9990"
      - "9993:9993"
      - "11112:11112"
      - "2762:2762"
      - "2575:2575"
      - "12575:12575"
    environment:
      WILDFLY_ADMIN_USER: admin
      WILDFLY_ADMIN_PASSWORD: admin
      POSTGRES_DB: pacsdb
      POSTGRES_USER: pacs
      POSTGRES_PASSWORD: pacs
      WILDFLY_DEPLOY_UI: "false"
      WILDFLY_PACSDS_USE_CCM: "false"
      WILDFLY_WAIT_FOR: ldap:389 db:5432
      WILDFLY_EXECUTER_MAX_THREADS: 200
      WILDFLY_CHOWN: /storage
      JAVA_OPTS: -Xms1g -Xmx4g -XX:MetaspaceSize=512m -XX:MaxMetaspaceSize=1g -XX:+UseG1GC -XX:MaxGCPauseMillis=100 -XX:G1HeapRegionSize=32M -Djboss.tx.node.id=arc -Djava.net.preferIPv4Stack=true -Djboss.modules.system.pkgs=org.jboss.byteman -Djava.awt.headless=true -agentlib:jdwp=transport=dt_socket,address=*:8787,server=y,suspend=n
      # JAVA_OPTS: -Xms512m -Xmx2g -XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=512m -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -XX:G1HeapRegionSize=16M -Djboss.tx.node.id=arc -Djava.net.preferIPv4Stack=true -Djboss.modules.system.pkgs=org.jboss.byteman -Djava.awt.headless=true -agentlib:jdwp=transport=dt_socket,address=*:8787,server=y,suspend=n
    depends_on:
      - ldap
      - db
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
      - /var/local/dcm4chee-arc/wildfly:/opt/wildfly/standalone
      - /var/local/dcm4chee-arc/storage:/storage
    networks:
      - pacs-network
networks:
  pacs-network:
    name: pacs-network
    driver: bridge
```
### _Record creation_

```console
docker run --rm --network=host dcm4che/dcm4che-tools:5.30.0 storescu -cDCM4CHEE@192.168.1.6:11112 /opt/dcm4che/etc/testdata/dicom
```


### _log details validation_

```console
tail -f /var/local/dcm4chee-arc/wildfly/log/server.log
2022-06-09 14:03:58,946 INFO  [org.dcm4che3.net.Connection] (EE-ManagedExecutorService-default-Thread-2) Start TCP Listener on /0.0.0.0:11112
2022-06-09 14:03:58,946 INFO  [org.dcm4che3.net.Connection] (EE-ManagedExecutorService-default-Thread-1) Start TCP Listener on /0.0.0.0:2575
2022-06-09 14:03:59,048 INFO  [org.dcm4che3.net.Connection] (EE-ManagedExecutorService-default-Thread-3) Start TCP Listener on /0.0.0.0:12575
2022-06-09 14:03:59,048 INFO  [org.dcm4che3.net.Connection] (EE-ManagedExecutorService-default-Thread-4) Start TCP Listener on /0.0.0.0:2762
2022-06-09 14:03:59,211 INFO  [org.jboss.as.server] (ServerService Thread Pool -- 46) WFLYSRV0010: Deployed "dcm4chee-arc-ui2-5.31.2-secure.war" (runtime-name : "dcm4chee-arc-ui2-5.31.2-secure.war")
2022-06-09 14:03:59,211 INFO  [org.jboss.as.server] (ServerService Thread Pool -- 46) WFLYSRV0010: Deployed "dcm4chee-arc-ear-5.31.2-psql-secure.ear" (runtime-name : "dcm4chee-arc-ear-5.31.2-psql-secure.ear")
2022-06-09 14:03:59,238 INFO  [org.jboss.as.server] (Controller Boot Thread) WFLYSRV0212: Resuming server
2022-06-09 14:03:59,246 INFO  [org.jboss.as] (Controller Boot Thread) WFLYSRV0025: WildFly Full 27.0.1.Final (WildFly Core 18.1.1.Final) started in 10685ms - Started 2819 of 3031 services (446 services are lazy, passive or on-demand) - Server configuration file in use: dcm4chee-arc-oidc.xml
2022-06-09 14:03:59,248 INFO  [org.jboss.as] (Controller Boot Thread) WFLYSRV0062: Http management interface listening on http://0.0.0.0:9990/management and https://0.0.0.0:9993/management
2022-06-09 14:03:59,248 INFO  [org.jboss.as] (Controller Boot Thread) WFLYSRV0053: Admin console listening on http://0.0.0.0:9990 and https://0.0.0.0:9993

```

### _Console URLs_

```console
http://192.168.1.6:9990
http://192.168.1.6:8080/weasis-pacs-connector/weasis?patientID=583295
```

### _weasis viewer_

```console
weasis://$dicom:rs --url "http://54.169.118.34:8080/dcm4chee-arc/aets/DCM4CHEE/rs" -r "patientID=583295" --query-ext "&includedefaults=false"
weasis://$dicom:rs --url "http://sbmch.org.in:8080/dcm4chee-arc/aets/DCM4CHEE/rs" -r "studyUID=1.3.51.0.7.11194864995.55786.31298.42464.47635.50239.12053" --query-ext "&includedefaults=false"
```

### _Endpoint URLs_

```console
1. DICOM QIDO-RS Base URL: http://192.168.1.6:8080/dcm4chee-arc/aets/DCM4CHEE/rs
2. DICOM STOW-RS Base URL: http://192.168.1.6:8080/dcm4chee-arc/aets/DCM4CHEE/rs
3. DICOM WADO-RS Base URL: http://192.168.1.6:8080/dcm4chee-arc/aets/DCM4CHEE/rs
4. DICOM WADO-URI: http://192.168.1.6:8080/dcm4chee-arc/aets/DCM4CHEE/wado
5. IHE XDS-I Retrieve Imaging Document Set: http://192.168.1.6:8080/dcm4chee-arc/xdsi/ImagingDocumentSource
6. REST-API-URL: https://petstore.swagger.io/index.html?url=https://dcm4che.github.io/dcm4chee-arc-light/swagger/openapi.json#/
```

## _demo case_

```bash
weasis://$dicom:rs --url "https://demo.orthanc-server.com/dicom-web" -r "patientID=ozp00SjY2xG"
```

### Docs

1. [dcm4chee-arc-light](https://github.com/dcm4che/dcm4chee-arc-light/wiki/Get-Started-Tutorials)
2. [register-curl-as-oidc-client-in-keycloak](https://github.com/dcm4che/dcm4chee-arc-light/wiki/Get-OIDC-Access-Token-using-curl#register-curl-as-oidc-client-in-keycloak)
3. [github-docs](https://github.com/orgs/januo-org/projects/1/views/9?pane=issue&itemId=45943850)
