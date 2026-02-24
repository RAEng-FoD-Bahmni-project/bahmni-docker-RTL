FROM bahmni/openmrs:1.0.0

# docker compose stop openmrs
# docker compose build openmrs
# docker compose up -d openmrs

USER root

# Comment
RUN echo 'running some openmrs commands'

# RUN rm ${OPENMRS_APPLICATION_DATA_DIRECTORY}/modules/iplitdistro*.omod

RUN rm ${OPENMRS_APPLICATION_DATA_DIRECTORY}/modules/operationtheater*.omod

COPY openmrs/resources/iplitdistro*.omod ${OPENMRS_APPLICATION_DATA_DIRECTORY}/modules/

COPY openmrs/resources/operationtheater*.omod ${OPENMRS_APPLICATION_DATA_DIRECTORY}/modules/

RUN mkdir -p /home/bahmni/uploaded-files
RUN setfacl -d -m o::rx -m g::rx /home/bahmni/uploaded-files/