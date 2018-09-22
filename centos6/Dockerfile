FROM juliogonzalez/centos6-postgresql:<!POSTRESQL_VER!>
MAINTAINER Julio Gonzalez Gil <git@juliogonzalez.es>

USER root

# Set locale (will be used by PostgreSQL initdb later on)
RUN localedef -i en_US -f UTF-8 en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LC_ALL en_US.UTF-8

# Create the wrapper to check for PostgreSQL updates
ADD files/check_updates.sh /opt/check_updates.sh

# PostgreSQL arguments
# Default version
ARG POSTGRESQL_VER=9.5
ENV POSTGRESQL_VER=${POSTGRESQL_VER}

# Pass the output of date command as DATE argument if you want
# make sure that the image is generated using the last
# packages (cache will be used for previous steps)
ARG DATE=None

# Generate a list of original packages and add EPEL repository ${DATE}
RUN rpm -qa --qf "%{NAME}\n" > /opt/packages-image.txt

# Install requirements for tds_fdw, psycopg2 and pymssql
RUN yum -y -q install epel-release && \
    yum -y -q install \
      automake \
      freetds \
      freetds-devel \
      gcc \
      git \
      python-devel \
      python-pip \
      rpm-build \
      sudo && \
    PATH=${PATH}:/usr/pgsql-${POSTGRESQL_VER}/bin && \
    pip -q install setuptools_git==1.1 && \
    pip -q install psycopg2 && \
    # Not in the official doc, but >= 2.1.4 does not work with Python 2.6
    pip -q install pymssql==2.1.3 && \
    yum -y -q remove \
     epel-release \
     python-devel \
     python-pip && \
    yum -q clean all

# Configure sudo
RUN echo -e 'postgres ALL=(ALL) NOPASSWD:ALL\nDefaults:postgres !requiretty' > /etc/sudoers.d/postgresql && \
    chmod 440 /etc/sudoers.d/postgresql

# Configure locales.conf if PostgreSQL < 9.4
ADD files/locales.conf /tmp/locales.conf
RUN /bin/bash -c "if (( $(echo ${POSTGRESQL_VER} 9.4 | awk '{print ($1 < $2)}') )); then \
                    mv /tmp/locales.conf /etc/locales.conf; \
                  else \
                    rm -f /tmp/locales.conf; \
                  fi"

USER postgres

CMD ["/bin/bash", "/opt/start_postgresql.sh"]
