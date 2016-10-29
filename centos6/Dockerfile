FROM juliogonzalez/centos6-postgresql:<!POSTRESQL_VER!>
MAINTAINER Julio Gonzalez Gil <git@juliogonzalez.es>

USER root

# Set locale (will be used by PostgreSQL initdb later on)
RUN localedef -i en_US -f UTF-8 en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LC_ALL en_US.UTF-8

# Create the wrapper to check for PostgreSQL updates
ADD files/check_updates.sh /opt/check_updates.sh

# Pass the output of date command as DATE argument if you want
# make sure that the image is generated using the last
# packages (cache will be used for previous steps)
ARG DATE=None

# Generate a list of original packages and add EPEL repository
RUN rpm -qa --qf "%{NAME}\n" > /opt/packages-image.txt && \
    yum -y -q install epel-release; yum -q clean all # ${DATE}

# Install requirements for tds_fdw (except PostgreSQL)
RUN yum -y -q install \
      automake \
      freetds \
      freetds-devel \
      gcc \
      git \
      python-devel \
      python-pip \
      rpm-build \
      sudo && \
    yum -q clean all

# Configure sudo
RUN echo -e 'postgres ALL=(ALL) NOPASSWD:ALL\nDefaults:postgres !requiretty' > /etc/sudoers.d/postgresql && \
    chmod 440 /etc/sudoers.d/postgresql

# PostgreSQL arguments
# Default version
ARG POSTGRESQL_VER=9.5
ENV POSTGRESQL_VER=${POSTGRESQL_VER}

# Configure locales.conf if PostgreSQL < 9.4
ADD files/locales.conf /tmp/locales.conf
RUN /bin/bash -c "if (( $(echo ${POSTGRESQL_VER} 9.4 | awk '{print ($1 < $2)}') )); then \
                    mv /tmp/locales.conf /etc/locales.conf; \
                  else \
                    rm -f /tmp/locales.conf; \
                  fi"

# Install psycopg2 and pymssql
RUN PATH=${PATH}:/usr/pgsql-${POSTGRESQL_VER}/bin && \
    pip install -q psycopg2 pymssql && \
    yum -y -q remove \
     python-devel \
     python-pip && \
    yum -q clean all

USER postgres

CMD ["/bin/bash", "/opt/start_postgresql.sh"]
