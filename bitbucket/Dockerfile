FROM krallin/centos-tini:centos7
LABEL author="Subatomic - https://github.com/absa-subatomic"

# https://confluence.atlassian.com/display/BitbucketServer/Bitbucket+Server+home+directory
ENV BITBUCKET_HOME          /var/atlassian/application-data/bitbucket
ENV BITBUCKET_INSTALL_DIR   /opt/atlassian/bitbucket

ENV UPDATE_VERSION=8u181
ENV JAVA_VERSION=1.8.0_181
ENV BUILD=b13
ENV HASH 96a7b8442fe848ef90c96a2fad6ed6d1
RUN yum -y install wget curl tar createrepo openssl && \
  wget -c --no-cookies --no-check-certificate \
  --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" \
    "http://download.oracle.com/otn-pub/java/jdk/${UPDATE_VERSION}-${BUILD}/${HASH}/jdk-${UPDATE_VERSION}-linux-x64.rpm" \
  --output-document="jdk-${UPDATE_VERSION}-linux-x64.rpm" && \
  rpm -i jdk-${UPDATE_VERSION}-linux-x64.rpm && \
  alternatives --install /usr/bin/java java /usr/java/jdk${JAVA_VERSION}/bin/java 1 && \
  alternatives --set java /usr/java/jdk${JAVA_VERSION}/bin/java && \
  export JAVA_HOME=/usr/java/jdk${JAVA_VERSION}/ && \
  echo "export JAVA_HOME=/usr/java/jdk${JAVA_VERSION}/" | tee /etc/environment && \
  source /etc/environment && \
  rm jdk-${UPDATE_VERSION}-linux-x64.rpm && \
  yum clean all && \
  rm -rf /var/cache/yum
ENV JAVA_HOME=/usr/java/jdk${JAVA_VERSION}-amd64
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

ARG BITBUCKET_VERSION=5.14.1
ARG DOWNLOAD_URL=https://downloads.atlassian.com/software/stash/downloads/atlassian-bitbucket-${BITBUCKET_VERSION}.tar.gz
COPY . /tmp
RUN curl -L --silent ${DOWNLOAD_URL} | tar -xz --strip-components=1 -C "$BITBUCKET_INSTALL_DIR"

ADD https://raw.githubusercontent.com/absa-subatomic/local-hadron-collider/master/minishift-addons/subatomic/certs/subatomic.ca.crt /etc/ca.cert.pem
RUN keytool -importcert \
  -file /etc/ca.cert.pem \
  -alias subatomic.local \
  -keystore $JAVA_HOME/jre/lib/security/cacerts \
  -storepass changeit \
  -noprompt

USER 1001
