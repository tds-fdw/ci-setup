FROM docker.io/juliogonzalez/ubuntu24.04-postgresql:<!POSTRESQL_VER!>
MAINTAINER Julio Gonzalez Gil <git@juliogonzalez.es>

USER root

# Create the wrapper to check for PostgreSQL updates
ADD files/check_updates.sh /opt/check_updates.sh

# PostgreSQL arguments
# Default version
ARG POSTGRESQL_VER=9.5
ENV POSTGRESQL_VER=${POSTGRESQL_VER}

# Pass the output of date command as DATE argument if you want
# make sure that the image is generated using the lastest
# packages (cache will be used for previous steps)
ARG DATE=None

# Generate a list of original packages ${DATE}
RUN dpkg --get-selections | grep 'install' | grep -v 'deinstall'|cut -f1 > /opt/packages-image.txt

# Install requirements for tds_fdw, psycopg2 and pymssql
RUN apt-get -qq -o=Dpkg::Use-Pty=0 update > /dev/null && \
    DEBIAN_FRONTEND=noninteractive apt-get -qq -o=Dpkg::Use-Pty=0 -y install \
     freetds-dev \
     freetds-common \
     gcc \
     git \
     libkrb5-dev \
     libsybdb5 \
     make \
     python3 \
     python3-psycopg2 \
     python3-pymssql \
     sudo > /dev/null && \
    apt-get -qq -o=Dpkg::Use-Pty=0 clean > /dev/null

# Configure sudo
RUN echo 'postgres ALL=(ALL) NOPASSWD:ALL\nDefaults:postgres !requiretty' > /etc/sudoers.d/postgresql && \
    chmod 440 /etc/sudoers.d/postgresql

# Configure locales.conf if PostgreSQL < 9.4
ADD files/locales.conf /tmp/locales.conf
RUN /bin/bash -c "if (( $(echo ${POSTGRESQL_VER} 9.4 | awk '{print ($1 < $2)}') )); then \
                    mv /tmp/locales.conf /etc/freetds/locales.conf; \
                  else \
                    rm -f /tmp/locales.conf; \
                  fi"

USER postgres

CMD ["/bin/bash", "/opt/start_postgresql.sh"]
