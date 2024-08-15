FROM docker.io/mimove/tickettrackr-extract-load-base-dependecies:1 

ARG BUILD_DATE
ARG VCS_REF
LABEL org.label-schema.build-date=${BUILD_DATE} \
        org.label-schema.name="tickettrackr"\
        org.label-schema.vcs-ref=${VCS_REF} \
        org.label-schema.license=MIT

# Install tesseract-ocr and poppler-utils
RUN apt-get update && \
    apt-get install -y tesseract-ocr poppler-utils && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*


COPY docker/scripts /app/scripts
COPY ./cli /app/cli
COPY ./requirements /app/requirements

WORKDIR /app/cli

ENV PYTHONPATH=/app/cli
ENV PYTHONUNBUFFERED=1

####################
## continue with the rest of the Dockerfile
## what/ how to run docker with .env file
####################

