FROM krallin/centos-tini:centos7
LABEL author="Subatomic - https://github.com/absa-subatomic"

# https://confluence.atlassian.com/display/BitbucketServer/Bitbucket+Server+home+directory
ENV BITBUCKET_HOME          /var/atlassian/application-data/bitbucket
ENV BITBUCKET_INSTALL_DIR   /opt/atlassian/bitbucket

ENV JAVA_VERSION=1.8.0_191

RUN yum -y install wget curl tar createrepo openssl java-1.8.0-openjdk && \

  source /etc/environment && \
  yum clean all && \
  rm -rf /var/cache/yum

ENV JAVA_HOME=/usr/lib/jvm/jre
RUN echo $JAVA_HOME
ENV PATH=$JAVA_HOME/bin:$PATH

RUN rpm -Uvh http://repo.iotti.biz/CentOS/5/noarch/lux-release-0-1.noarch.rpm && \
  rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-LUX && \
  yum -y install git && \
  yum clean all && \
  rm -rf /var/cache/yum

RUN mkdir -p ${BITBUCKET_HOME} \
    && chmod g+rwX ${BITBUCKET_HOME}/
VOLUME ["${BITBUCKET_HOME}"]

# Expose HTTP and SSH ports
EXPOSE 7990
EXPOSE 7999

WORKDIR $BITBUCKET_HOME

CMD ["/entrypoint.sh", "-fg"]
ENTRYPOINT ["/usr/bin/tini", "--"]

ADD https://raw.githubusercontent.com/absa-subatomic/local-hadron-collider/master/minishift-addons/subatomic/certs/subatomic.ca.crt /etc/pki/ca-trust/source/anchors/subatomic-ca-chain.crt
RUN chown 1001:0 /etc/pki/ca-trust/source/anchors/subatomic-ca-chain.crt && \
    update-ca-trust

COPY bitbucket.properties ${BITBUCKET_INSTALL_DIR}/bitbucket.properties

COPY entrypoint.sh /entrypoint.sh
RUN chmod g+x /entrypoint.sh

RUN mkdir -p ${BITBUCKET_INSTALL_DIR} \
    && chmod g+rwX ${BITBUCKET_INSTALL_DIR}

ARG BITBUCKET_VERSION=5.4.6
ARG DOWNLOAD_URL=https://downloads.atlassian.com/software/stash/downloads/atlassian-bitbucket-${BITBUCKET_VERSION}.tar.gz
COPY . /tmp
RUN curl -L --silent ${DOWNLOAD_URL} | tar -xz --strip-components=1 -C "$BITBUCKET_INSTALL_DIR"

ADD https://raw.githubusercontent.com/absa-subatomic/local-hadron-collider/master/minishift-addons/subatomic/certs/subatomic.ca.crt /etc/ca.cert.pem
RUN keytool -importcert \
  -file /etc/ca.cert.pem \
  -alias subatomic.local \
  -keystore $JAVA_HOME/lib/security/cacerts \
  -storepass changeit \
  -noprompt

USER 1001
