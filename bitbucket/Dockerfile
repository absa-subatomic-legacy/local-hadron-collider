FROM openjdk:8u121-alpine
LABEL author="Subatomic - https://github.com/absa-subatomic"

# https://confluence.atlassian.com/display/BitbucketServer/Bitbucket+Server+home+directory
ENV BITBUCKET_HOME          /var/atlassian/application-data/bitbucket
ENV BITBUCKET_INSTALL_DIR   /opt/atlassian/bitbucket

RUN mkdir -p ${BITBUCKET_HOME} \
    && chmod g+rwX ${BITBUCKET_HOME}/
VOLUME ["${BITBUCKET_HOME}"]

# Expose HTTP and SSH ports
EXPOSE 7990
EXPOSE 7999

WORKDIR $BITBUCKET_HOME

CMD ["/entrypoint.sh", "-fg"]
ENTRYPOINT ["/sbin/tini", "--"]

RUN apk add --no-cache ca-certificates wget curl git openssh bash procps openssl perl ttf-dejavu tini
ADD https://raw.githubusercontent.com/absa-subatomic/local-hadron-collider/master/minishift-addons/subatomic/certs/subatomic.ca.crt /usr/local/share/ca-certificates/ca.cert.crt
RUN update-ca-certificates

COPY bitbucket.properties ${BITBUCKET_INSTALL_DIR}/bitbucket.properties

COPY entrypoint.sh /entrypoint.sh
RUN chmod g+x /entrypoint.sh

RUN mkdir -p ${BITBUCKET_INSTALL_DIR} \
    && chmod g+rwX ${BITBUCKET_INSTALL_DIR}

ARG BITBUCKET_VERSION=5.12.2
ARG DOWNLOAD_URL=https://downloads.atlassian.com/software/stash/downloads/atlassian-bitbucket-${BITBUCKET_VERSION}.tar.gz
COPY . /tmp

RUN curl -L --silent ${DOWNLOAD_URL} | tar -xz --strip-components=1 -C "$BITBUCKET_INSTALL_DIR"

USER 1001