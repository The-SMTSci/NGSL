##############################################################################
# Dockerfile - with ubuntu 18.04 for better support of ancillary programs
# and postgres 11.7 to reflect the development environment of this container
# set the environmnet to support NGSL work.
#
# user  ngsl, password ngsl
# postgres role ngsl, password ngsl
#
# requires PMachine supply disk space for the database:
#    
#    /etc/postgresql
#    /var/log/postgresql
#    /var/lib/postgresql
# 
# Assume PostgreSQL is installed on the PMachine, using the usual resources
# This CMachine wants
# 
# PMachine> docker build -t ngslv1 . # NOTE trailing dot!
# 
# docker run --rm -P --name myngsl ngslv1
# 
# PMachine> docker ps | grep myngsl the port values etc. 
# PMachine> psql -p 55432 -h 0.0.0.0 -d ngsl -u ngsl --password
# 
#############################################################################

FROM ubuntu:18.04
LABEL maintainer="wg@stmsci.net"
LABEL author="Wayne Green"

RUN apt update

# Load up the stuff we want in this container
# vi vi editor and locate for easier sysadmin
# for i-386
#RUN dpkg --add-architecture i386         && \
#apt-get update                           && \
#apt-get install -y libstdc++6:i386       && \
#RUN apt-get install curl ca-certificates gnupg

RUN apt-get install -y vim                && \
    apt-get install -y curl               && \
    apt-get install -y ca-certificates    && \
    apt-get install -y gnupg              && \
    apt-get install -y --reinstall procps && \
    apt-get install -y net-tools          && \
    apt-get install -y iproute2           && \
    apt-get install -y rsyslog            && \
    apt-get install -y openssh-server

RUN useradd -m -d /home/ngsl -G dialout -p "$(openssl passwd -1 ngsl)" ngsl

RUN curl https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

# Add PostgreSQL's repository stable release of PostgreSQL 11.7.
RUN echo "deb http://apt.postgresql.org/pub/repos/apt/ precise-pgdg main" > /etc/apt/sources.list.d/pgdg.list

# Install ``python-software-properties``, ``software-properties-common`` and PostgreSQL 11.7
#         ``software-properties-common``
#         PostgreSQL 11.7
#  There are some warnings (in red) that show up during the build. You can hide
#  them by prefixing each apt-get statement with DEBIAN_FRONTEND=noninteractive
RUN apt-get install -y python-software-properties \
                       software-properties-common \
                       postgresql-11.7 \
                       postgresql-client-11.7 \
                       postgresql-contrib-11.7

# Note: The official Debian and Ubuntu images automatically ``apt-get clean``
# after each ``apt-get``

# Run rest of commands as ``postgres`` user when installed by ``apt-get installed``
USER postgres

# Create a PostgreSQL role named ``ngsl`` with ``ngsl`` as the password and
# then create a database `ngsl` owned by the ``ngsl`` role.
RUN    /etc/init.d/postgresql start &&\
    psql --command "CREATE USER ngsl WITH SUPERUSER PASSWORD 'ngsl';" &&\
    createdb -O ngsl ngsl

# PostgreSQL pg_hba.conf to permit remote connections
RUN echo "host all  all    0.0.0.0/0  md5" >> /etc/postgresql/11.7/main/pg_hba.conf

# PostgreSQL listen_addresses...
RUN echo "listen_addresses='*'" >> /etc/postgresql/11.7/main/postgresql.conf

# Expose the PostgreSQL port
EXPOSE 5432

# Add VOLUMEs to allow backup of config, logs and databases
VOLUME  ["/etc/postgresql", "/var/log/postgresql", "/var/lib/postgresql"]

# Set the default command to run when starting the container
CMD ["/usr/lib/postgresql/11.7/bin/postgres", \
    "-D", "/var/lib/postgresql/11.7/main", \
    "-c", "config_file=/etc/postgresql/11.7/main/postgresql.conf"]
