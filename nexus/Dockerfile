FROM centos:centos7

ENV SONATYPE_WORK=/sonatype-work \
    JAVA_HOME=/usr/lib/jvm/jre

RUN yum install -y \
  curl tar createrepo java-1.8.0-openjdk \
  && yum clean all

ARG NEXUS_VERSION=2.14.9-01
ARG NEXUS_DOWNLOAD_URL=https://download.sonatype.com/nexus/oss/nexus-${NEXUS_VERSION}-bundle.tar.gz

RUN mkdir -p /opt/sonatype/nexus \
  && curl --fail --silent --location --retry 3 \
    ${NEXUS_DOWNLOAD_URL} \
  | gunzip \
  | tar x -C /tmp nexus-${NEXUS_VERSION} \
  && mv /tmp/nexus-${NEXUS_VERSION}/* /opt/sonatype/nexus/ \
  && rm -rf /tmp/nexus-${NEXUS_VERSION}

RUN chmod g+rwX /opt/sonatype/nexus \
  && mkdir -p ${SONATYPE_WORK} \
  && chmod g+rwX ${SONATYPE_WORK}

VOLUME ${SONATYPE_WORK}

EXPOSE 8081
WORKDIR /opt/sonatype/nexus
USER 1001
ENV CONTEXT_PATH /nexus
ENV MAX_HEAP 512m
ENV MIN_HEAP 256m
ENV JAVA_OPTS="-server -Djava.net.preferIPv4Stack=true"
ENV LAUNCHER_CONF="./conf/jetty.xml ./conf/jetty-requestlog.xml"
CMD ${JAVA_HOME}/bin/java \
  -Dnexus-work=${SONATYPE_WORK} -Dnexus-webapp-context-path=${CONTEXT_PATH} \
  -Xms${MIN_HEAP} -Xmx${MAX_HEAP} \
  -cp 'conf/:lib/*' \
  ${JAVA_OPTS} \
  org.sonatype.nexus.bootstrap.Launcher ${LAUNCHER_CONF}