# Run Elasticsearch with Kibana

General notes: it is useful here not to run both applications in `-d` mode in order to see what's going on

1. Get a working image of ES6.x
2. Run:
  ```
  docker run --rm --name=elasticsearch_6.5.4 -p 127.0.0.1:9200:9200 -e "discovery.type=single-node" -e "transport.host=127.0.01" --mount type=bind,source=[local path to your data],destination=/data --mount type=bind,source=/srv/docker/elasticsearch/elasticsearch.yml,destination=/elasticsearch/config/elasticsearch.yml [ES 6.5.4 image name]
  ```

  Notes:
  - `-p 127.0.0.1:9200:9200` option makes ES not visible to the outside
  - the first `--mount` option hooks in the docker volume, the second your configuration `.yml` file.
  - make sure the image expects the destination paths! Otherwise you will end up with non-persistent data.

  Example `elasticsearch.yml`:

  ```
  network:
    host: 0.0.0.0
  path:
    data: /data/data
    logs: /data/log
  ```

3. Run:
  ```
  docker run --name kibana_6.5.4 -p 127.0.0.1:5601:5601 \
  --link elasticsearch_6.5.4:elasticsearch_6.5.4 -e "ELASTICSEARCH_URL=http://172.17.0.2:9200" \
  docker.elastic.co/kibana/kibana:6.5.4
  ```

  Notes:
  - In Kibana, your ES will show up in "Yellow" state - this is normal, as [it's only green here when running in a replicated, e.g. two-clustered mode](http://chrissimpson.co.uk/elasticsearch-yellow-cluster-status-explained.html)
