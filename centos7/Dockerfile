FROM docker.io/juliogonzalez/centos7-postgresql:<!POSTRESQL_VER!>
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
# For centos-release-scl and llvm see https://github.com/tds-fdw/tds_fdw/blob/master/InstallCentOS.md#centos7-and-postgresql--11
RUN yum -y -q install centos-release-scl epel-release && \
    yum -y -q install \
      freetds \
      freetds-devel \
      gcc \
      git \
      llvm-toolset-7-clang \
      llvm5.0 \
      make \
      python2-certifi \
      python-devel \
      python-pip \
      rpm-build \
      sudo && \
    PATH=${PATH}:/usr/pgsql-${POSTGRESQL_VER}/bin && \
    pip -q install setuptools_git && \
    pip -q install Cython==0.29.36 && \
    # psycopg2 > 2.8.6 only sypports python >= 3.6
    # https://www.psycopg.org/articles/2021/06/16/psycopg-29-released/
    pip -q install psycopg2==2.8.6 && \
    # pymssql > 2.1.5 dropped support for Python 2
    # https://github.com/pymssql/pymssql/blob/master/ChangeLog.rst#version-220---2021-04-08---mikhail-terekhov
    # Prebuilt pymssql does not work: https://www.juliogonzalez.es/fixing-pymssql-2-1-5-from-pip-on-centos7/501
    pip -q install --no-binary :all: pymssql==2.1.5 && \
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
