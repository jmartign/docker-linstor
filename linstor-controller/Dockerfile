FROM ubuntu:bionic
ARG CI_COMMIT_REF_NAME=
ENV DEBIAN_FRONTEND noninteractive

# Install repos and system upgrade
RUN apt-get -y update \
 && apt-get install -y wget gnupg \
 && apt-key adv --keyserver keyserver.ubuntu.com --recv-keys CC1B5A793C04BB3905AD837734893610CEAA9512 \
 && echo "deb http://ppa.launchpad.net/linbit/linbit-drbd9-stack/ubuntu bionic main" \
      > /etc/apt/sources.list.d/linbit.list \
 && apt-get update \
 && apt-get -y upgrade \
 && apt-get clean

# Install tini
ENV TINI_VERSION v0.18.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /tini
RUN chmod +x /tini
ENTRYPOINT ["/tini", "--"]

# Install linstor-controller
RUN apt-get update \
 && apt-get install -y linstor-controller linstor-client \
 && apt-get clean \
 && mkdir -p /config /logs \
 && /usr/share/linstor-server/bin/linstor-config create-db-file \
      /data/linstordb > /config/database.cfg

# Install postgresql connector
ENV JDBC_POSTGRESQL_VERSION 42.2.2
RUN wget https://jdbc.postgresql.org/download/postgresql-${JDBC_POSTGRESQL_VERSION}.jar \
      -O /usr/share/linstor-server/lib/postgresql-${JDBC_POSTGRESQL_VERSION}.jar

# Downgrade flyway (https://github.com/LINBIT/linstor-server/issues/34)
ENV JDBC_FLYWAY_VERSION 5.0.7
RUN rm -rf /usr/share/linstor-server/lib/flyway-core-* \
 && wget https://repo1.maven.org/maven2/org/flywaydb/flyway-core/${JDBC_FLYWAY_VERSION}/flyway-core-${JDBC_FLYWAY_VERSION}.jar \
      -O /usr/share/linstor-server/lib/flyway-core-${JDBC_FLYWAY_VERSION}.jar

CMD [ "/usr/share/linstor-server/bin/Controller", "--logs=/logs", "--config-directory=/config" ]
