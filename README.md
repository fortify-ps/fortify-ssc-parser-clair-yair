# Fortify SSC Parser Plugin for Clair / Yair

* Travis-CI builds: https://travis-ci.com/fortify-ps/fortify-ssc-parser-clair-yair
* Binaries: https://bintray.com/beta/#/fortify-ps/binaries/fortify-ssc-parser-clair-yair?tab=files
* Sample JSON file: https://github.com/fortify-ps/fortify-ssc-parser-clair-yair/tree/1.0-SNAPSHOT/src/test/resources

This Fortify SSC parser plugin allows for importing Clair scan results from JSON reports generated by Yair. See the following links for more information about Clair and Yair:

* Yair GitHub repository (master branch): https://github.com/yfoelling/yair
* Clair GitHub repository (latest 2.x version as supported by Yair): https://github.com/quay/clair/tree/v2.1.2
* Legacy Clair documentation: https://coreos.com/clair/docs/latest/ 

TODO: Provide further information and instructions

TODO: Provide comparison between the various Clair-related parser plugins (fortify-ssc-parser-clair-rest, fortify-ssc-parser-clair-yair, ...)


### Generate Clair JSON file

The following steps were used to generate the `src/test/resources/node_10.14.2-jessie.yair.json` file:

```bash

# Start Postgres DB without superuser password (for testing only)
docker run -e POSTGRES_HOST_AUTH_METHOD=trust --name postgres -p 5432:5432 -d postgres

# Check Postgres started OK
docker logs postgres

# Create and navigate into clair directory
mkdir clair
cd clair

# Get sample config, save as clair.config
curl -L https://raw.githubusercontent.com/coreos/clair/master/config.yaml.sample -o $PWD/clair.config

# Update config file to use postgres docker container
sed -i 's/source: host=localhost/source: host=postgres/g' $PWD/clair.config

# Run clair as dameon
docker run --name clair --link postgres:postgres -p 6060:6060 -p 6061:6061 -v $PWD/clair.config:/config/config.yaml -d quay.io/coreos/clair:latest -config=/config/config.yaml

# Check Clair started OK
docker logs clair

cat <<'EOF' > $PWD/yair.config
---
registry:
  host: "registry.hub.docker.com"

clair:
  host: "clair:6060"

output:
  format: json

fail_on:
  score: 0
  big_vulnerability: false
EOF

# Analyze the node:10.14.2-jessie image and save results in JSON file
docker run -v $PWD/yair.config:/opt/yair/config/config.yaml:ro --link clair:clair yfoelling/yair node:10.14.2-jessie > node_10.14.2-jessie.yair.json

```