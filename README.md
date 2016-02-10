# Cowhand - Rancher Monitoring and Alerting

## How to Use

   * Add `https://github.com/danieldent/cowhand` as a Rancher Catalog under Admin -> Settings.
   * Launch the `cowhand`, `cowhand-cadvisor`, and `cowhand-node-exporter services`.
   * You'll want to bake an `alertmanager` image to configure your alerting targets and templates. Example `Dockerfile`:

```Dockerfile
    FROM prom/alertmanager:0.1.0-beta2
    COPY alertmanager.yml /etc/alertmanager/config.yml
```

See https://github.com/prometheus/alertmanager/ for an example configuration.

   * Create a Rancher Load Balancer or use another technique to get access to the following services on the following ports:
     * cowhand/influxdb 8083
     * cowhand/grafana 3000
     * cowhand/prometheus-rancher 9090
     * cowhand/alertmanager 9093
   * When adding data sources to Grafana, configure Grafana to proxy access to the datasources.

## Monitoring Additional Services

[Prom-rancher-sd](https://github.com/DanielDent/prom-rancher-sd) is used to do automatic service discovery of Rancher services into Prometheus.
Simply add the `com.danieldent.cowhand.prom.port` label to a service to specify the port on which Prometheus should scrape `/metrics`.

## Future Ideas

  * Elasticsearch/Logstash/Kibana integration(s)
  * Consider adding support for https://github.com/JamesBarwell/prometheus-rancher-exporter. Undocumented Rancher labels are available to automatically provide API keys to a service designated as a system service.

## Known Issues

  * Google cadvisor has issues with ZFS. See https://github.com/google/cadvisor/pull/1020#issuecomment-181153825.
  * Launching our own cadvisor instances is a waste of system resources - Rancher already creates cadvisor instances on every node. However, these instances do not appear to be accessible over the Rancher overlay network. Additionally, it is unclear if we could arrange for those instances to push metrics into InfluxDB.
  * InfluxDB appears to function but it has not been tested yet.
  * prometheus-rancher gives error messages such as `level=error msg="Error reading file \"/prom-rancher-sd-data/rancher.json\": unexpected end of JSON input" source="file.go:196"`. I have not investigated to understand why. It's possible it would be desireable to create the file elsewhere on the disk and then move it into the data volume (so that updates to the file are atomic). It may be a JSON formatting issue. It may be something else entirely.'


## Desired Pull Requests

  * Automatically set InfluxDB and Prometheus as Grafana data sources
  * Arrange for Prometheus to use a service discovery approach to become aware of the Alertmanager, so that having an Alertmanager running is optional. Using the Rancher metadata service (as is done by [Prom-rancher-sd](https://github.com/DanielDent/prom-rancher-sd)) to decide on the command line used to start Prometheus and then watching the metadata service for changes would be one way to implement this.
  * Pre-configured Grafana dashboards for commonly monitored metrics
  * Icon filess for each of the services under the templates folder
  * Web proxy which authenticates users and provides access to Prometheus, Influxdb, Grafana, and Alertmanager on a single domain
    * Premethus uses the -web.external-url option on the command line, while Grafana uses the GF_SERVER_ROOT_URL environment variable. Alertmanager also has a command line option. I have not investigated the approach for InfluxDB.
  * Other pull requests also welcome

## Further Reading

   * http://rancher.com/docker-monitoring-continued-prometheus-and-sysdig/
   * https://www.brianchristner.io/how-to-setup-docker-monitoring/
   * https://www.ctl.io/developers/blog/post/monitoring-docker-services-with-prometheus/

## License

Copyright 2016 [Daniel Dent](https://www.danieldent.com/).

Licensed under the Apache License, Version 2.0 (the "License");
you may not use these files except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

Third-party contents included in builds of the image are licensed separately.
