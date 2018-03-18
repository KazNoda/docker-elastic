[![Build Status on Travis](https://travis-ci.org/shazChaudhry/docker-elastic.svg?branch=server.basePath "CI build status on Travis")](https://travis-ci.org/shazChaudhry/docker-elastic)

### User story
As a DevOps team member, I want to install [Elastic Stack](https://www.elastic.co/products) so that all application and system logs are collected centrally for searching, visualizing, analyzing and reporting purpose

<p align="center">
  <img src="./pics/monitoring.png" alt="monitoring" style="width: 250px;" /><br>
  <a href="https://www.elastic.co/blog/psd2-architectures-with-the-elastic-stack-part-ii">Monitoring Modern Banking API Architectures with the Elastic Stack, Part II</a>
</p>

<p align="center">
  <img src="./pics/elastic-stack.png" alt="Beats platform" style="width: 250px;"/>
  <img src="./pics/elastic-products.PNG" alt="Elastic products" style="width: 250px;"/>
  <br>
  <a href="https://www.elastic.co/guide/en/logstash/current/deploying-and-scaling.html">Deploying and Scaling Logstashedit</a>
</p>

<p align="center">
  <img src="./pics/cyber-webinar-thumbnail.jpeg" alt="Threat Detection" style="width: 250px;" /><br>
  <a href="https://www.elastic.co/webinars/security-and-threat-detection-with-the-elastic-stack">Security and Threat Detection with the Elastic Stack</a>
</p>

<p align="center">
  <img src="./pics/machine-learning.png" alt="Machine Learning" style="width: 250px;" /><br>
  <a href="https://www.elastic.co/webinars/security-and-threat-detection-with-the-elastic-stack">Machine Learning and Elasticsearch for Security Analytics</a>
</p>


### Assumptions
All containerized application services will start with [GELF](http://docs.graylog.org/en/2.2/pages/gelf.html) log driver in order to send logs to Elastic Stack

### Prerequisite
* Infrastructre is setup in [Docker swarm mode](https://docs.docker.com/engine/swarm/)
* On each cluster node, ensure maximum map count check _(required for Elasticsearch)_ is set: `sudo sysctl -w vm.max_map_count=262144`

### Installation instructions
* Login to the master node in your Docker Swarm cluster
* Clone this repo and change directory by following these commands
```
  alias git='docker run -it --rm --name git -v $PWD:/git -w /git indiehosters/git git'
  git version
  git clone https://github.com/shazChaudhry/docker-elastic.git
  sudo chown -R $USER docker-elastic
  cd docker-elastic
  ```
* Deploy Elasticsearch by running the following commands:
  * `export ELASTIC_VERSION=6.2.2`
  * `docker network create --driver overlay elastic`
  * `docker stack deploy --compose-file docker-compose.yml elastic`
* Check status of the stack services by running the following commands:
  * `docker stack services elastic`
  * `docker stack ps --no-trunc elastic` _(address any error reported at this point)_
  * `curl -XGET -u elastic:changeme 'localhost:9200/_cat/health?v&pretty'` _(Inspect status of cluster)_
* Once all services are running, execute the following commands:
  * `docker stack deploy --compose-file filebeat-docker-compose.yml filebeat`
  * Running the following command should produce elasticsearch index and one of the rows should have _filebeat-*_
    * `curl -XGET -u elastic:changeme 'localhost:9200/_cat/indices?v&pretty'`
  * `docker stack deploy --compose-file metricbeat-docker-compose.yml metricbeat`
  * Running the following command should produce elasticsearch index and one of the rows should have _metricbeat-*_
    * `curl -XGET -u elastic:changeme 'localhost:9200/_cat/indices?v&pretty'`

### Testing
* Wait until all stack services are up and running
* Run jenkins container on one of the Docker Swarm node as follows:
  * `docker container run -d --rm --name jenkins -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home --log-driver=gelf --log-opt gelf-address=udp://127.0.0.1:12201  jenkinsci/blueocean`
    * Note that _`--log-driver=gelf --log-opt gelf-address=udp://127.0.0.1:12201`_ sends container console logs to Elastic stack
* Login at `http://<master_node_ip>:5601` _(Kibana)_  which should show Management tab
  * username = `elastic`
  * password = `changeme`
* On the Kibana Management tab, configure an index pattern
  * Index name or pattern = `logstash-*`
  * Time-field name = `@timestamp`
* Click on Kibana Discover tab to view jenkins console logs

### References
- [Installing Elastic Stack](https://www.elastic.co/guide/en/elastic-stack/current/installing-elastic-stack.html)
- [Elastic Examples](https://github.com/elastic/examples)
- [ Machine Learning in the Elastic Stack - YouTube](https://www.youtube.com/watch?v=n6xW6YWYgs0&feature=youtu.be)
