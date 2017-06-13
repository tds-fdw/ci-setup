FROM juliogonzalez/ubuntu14.04-postgresql:<!POSTRESQL_VER!>
MAINTAINER Julio Gonzalez Gil <git@juliogonzalez.es>

USER root

# Create the wrapper to check for PostgreSQL updates
ADD files/check_updates.sh /opt/check_updates.sh

# Pass the output of date command as DATE argument if you want
# make sure that the image is generated using the lastest
# packages (cache will be used for previous steps)
ARG DATE=None

# Generate a list of original packages and install requirements for tds_fdw
RUN dpkg --get-selections | grep 'install' | grep -v 'deinstall'|cut -f1 > /opt/packages-image.txt && \
    apt-get -qq -o=Dpkg::Use-Pty=0 update > /dev/null && \
    DEBIAN_FRONTEND=noninteractive apt-get -qq -o=Dpkg::Use-Pty=0 -y install \
     freetds-dev \
     freetds-common \
     gcc \
     git \
     libsybdb5 \
     make \
     python \
     python-dev \
     python-pip \
     sudo > /dev/null && \
    apt-get -qq -o=Dpkg::Use-Pty=0 clean > /dev/null # ${DATE}

# Configure sudo
RUN echo 'postgres ALL=(ALL) NOPASSWD:ALL\nDefaults:postgres !requiretty' > /etc/sudoers.d/postgresql && \
    chmod 440 /etc/sudoers.d/postgresql

# PostgreSQL arguments
# Default version
ARG POSTGRESQL_VER=9.5
ENV POSTGRESQL_VER=${POSTGRESQL_VER}

# Configure locales.conf if PostgreSQL < 9.4
ADD files/locales.conf /tmp/locales.conf
RUN /bin/bash -c "if (( $(echo ${POSTGRESQL_VER} 9.4 | awk '{print ($1 < $2)}') )); then \
                    mv /tmp/locales.conf /etc/freetds/locales.conf; \
                  else \
                    rm -f /tmp/locales.conf; \
                  fi"

# Install psycopg2 and pymssql
RUN PATH=${PATH}:/usr/lib/postgresql/${POSTGRESQL_VER}/bin/ && \
    pip -q install psycopg2 pymssql && \
    apt-get -qq -o=Dpkg::Use-Pty=0 -y purge python-pip python-dev > /dev/null && \
    apt-get -qq -o=Dpkg::Use-Pty=0 -y --purge autoremove > /dev/null && \
    apt-get -qq -o=Dpkg::Use-Pty=0 clean > /dev/null

USER postgres

CMD ["/bin/bash", "/opt/start_postgresql.sh"]
