FROM docker.io/juliogonzalez/opensuseleap15.6-postgresql:<!POSTRESQL_VER!>
MAINTAINER Julio Gonzalez Gil <git@juliogonzalez.es>

USER root

# Set locale (will be used by PostgreSQL initdb later on)
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

# Generate a list of original packages ${DATE}
RUN rpm -qa --qf "%{NAME}\n" > /opt/packages-image.txt

# Install requirements for tds_fdw, psycopg2 and pymssql
RUN zypper -q install -y \
      clang \
      freetds \
      freetds-devel \
      gcc \
      git \
      make \
      python3 \
      python3-pymssql \
      python3-psycopg2 \
      rpm-build \
      sudo && \
    zypper clean -a

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
