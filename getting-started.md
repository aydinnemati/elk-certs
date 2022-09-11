# renew elasticsearch cluster (elasticsearch - kibana - logstach) certificates with new CA

1. first of all we should generate a new CA:
- create an instances.yml file for nodes in cluster like below example (**just add elasticsearch data nodes**)
```yaml
instances:
   - name: **<NODE NAME>**
     dns:
       - *<DOMAIN>*
     ip:
       - *<IP ADDRESS>*
.
.
.

```
- go to path **/usr/share/elasticsearch/bin** and generate new CA:
```bash
$ ./elasticsearch-certutil ca --silent --pem -out /<PATH OF OUTPUT FILE>/ca.zip
```
- extract ca certs from outputfile
- then generate certs of nodes from instances.yml file using our new CA:
```bash
$ ./elasticsearch-certutil cert --silent --pem --out /<PATH OF OUTPUT ZIP FILE OF NODE CERTIFICATES>/certs-a.zip --in /<PATH TO INSTANCES YML FILE>/instance.yml --ca-cert /<PATH TO CA CRT FILE>/ca.crt --ca-key /<PATH TO CA KEY FILE>/ca.key
```
- copy ca and each node certificates to every node
- modify configuration files of nodes and add lines below:
> - /etc/elasticsearch/elasticsearch.yml
```yaml
xpack.security.enabled: true
xpack.security.http.ssl.enabled: true
xpack.security.http.ssl.key: /<PATH OF NODE'S KEY FILE>.key
xpack.security.http.ssl.certificate: /<PATH OF NODES CRT FILE>.crt
xpack.security.http.ssl.certificate_authorities: /<PATH OF CA CRT FILE>/ca.crt
xpack.security.http.ssl.verification_mode: certificate
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.key: /<PATH OF NODE'S KEY FILE>.key
xpack.security.transport.ssl.certificate: /<PATH OF NODES CRT FILE>.crt
xpack.security.transport.ssl.certificate_authorities: /<PATH OF CA CRT FILE>/ca.crt
xpack.security.transport.ssl.verification_mode: certificate
```

> - /etc/kibana/kibana.yml
```yaml
elasticsearch.ssl.certificateAuthorities: [ "/<PATH TO CA CRT FILE>/ca.crt" ]
```

> - /etc/logstach/logstash.yml

```yaml
xpack.monitoring.elasticsearch.ssl.certificate_authority: "/etc/logstash/certs/ca.crt"
xpack.management.elasticsearch.ssl.certificate_authority: "/etc/logstash/certs/ca.crt"
```
2. RESTART NODES and enjoy :)