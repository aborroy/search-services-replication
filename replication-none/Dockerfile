ARG SEARCH_TAG
FROM alfresco/alfresco-search-services:${SEARCH_TAG}

# Configuration arguments passed from Docker Compose service definition
ARG SOLR_HOSTNAME
ARG ENABLE_MASTER
ARG ENABLE_SLAVE
ARG MASTER_HOST
ENV SOLR_HOSTNAME $SOLR_HOSTNAME
ENV ENABLE_MASTER $ENABLE_MASTER
ENV ENABLE_SLAVE $ENABLE_SLAVE
ENV MASTER_HOST $MASTER_HOST

# Base Docker Image for alfresco-search-services is exposing "$DIST_DIR/solrhome" as volume.
# This makes contents inside this folder immutable, so the files inside can't be modified
# in Dockerfile extensions. Because of this, "search_config_setup.sh" script is used in this
# Dockerfile to modify files under the folder, as the script is executed once the Container
# starts and changes are persisted.

# When Alfresco Search Services creates "alfresco" and "archive" SOLR Cores, configuration
# is copied from "rerank" template. So making changes in this template, generates both cores
# with the same configuration. If you are using volumes with existing cores, the changes below
# must be applied to configuration files on each node (instead on the template itself)

# Configure SOLR cores to run in HTTP mode
RUN sed -i '/^bash.*/i sed -i "'"s/alfresco.secureComms=https/alfresco.secureComms=none/g"'" ${DIST_DIR}/solrhome/templates/rerank/conf/solrcore.properties\n' \
    ${DIST_DIR}/solr/bin/search_config_setup.sh && \
# Configure Alfresco Service Name
    sed -i '/^bash.*/i sed -i "'"s/alfresco.host=localhost/alfresco.host=alfresco/g"'" ${DIST_DIR}/solrhome/templates/rerank/conf/solrcore.properties\n' \
    ${DIST_DIR}/solr/bin/search_config_setup.sh && \
# Set Hostname for this Node
    sed -i '/^bash.*/i sed -i "'"s/solr.host=localhost/solr.host=${SOLR_HOSTNAME}/g"'" ${DIST_DIR}/solrhome/conf/shared.properties\n' \
    ${DIST_DIR}/solr/bin/search_config_setup.sh && \
# Set Master / Slave configuration for this Node
    sed -i '/^bash.*/i echo "\nenable.master=${ENABLE_MASTER}\nenable.slave=${ENABLE_SLAVE}" >> ${DIST_DIR}/solrhome/templates/rerank/conf/solrcore.properties\n' \
    ${DIST_DIR}/solr/bin/search_config_setup.sh

# Apply configuration for Master SOLR Node
RUN if [ "$ENABLE_MASTER" = "true" ] ; then \
sed -i "/^bash.*/i sed -i '/^\\\\\s*<requestHandler name=\"\\\\/replication\".*/a \
    <lst name=\"master\">\
      <str name=\"replicateAfter\">commit</str>\
      <str name=\"replicateAfter\">startup</str>\
      <str name=\"confFiles\">schema.xml,stopwords.txt</str>\
    </lst>' ${DIST_DIR}/solrhome/templates/rerank/conf/solrconfig.xml\n" ${DIST_DIR}/solr/bin/search_config_setup.sh; \
# Apply configuration for Slave SOLR Node
else \
sed -i "/^bash.*/i sed -i '/^\\\\\s*<requestHandler name=\"\\\\/replication\".*/a \
    <lst name=\"slave\">\
      <str name=\"masterUrl\">http://${MASTER_HOST}:8983/solr/alfresco</str>\
      <str name=\"pollInterval\">00:00:60</str>\
    </lst>' ${DIST_DIR}/solrhome/templates/rerank/conf/solrconfig.xml\n" ${DIST_DIR}/solr/bin/search_config_setup.sh; \
fi
