FROM openjdk:8-alpine

ARG version=4.8.0
ARG user=rocketmq
ARG group=rocketmq
ARG uid=3000
ARG gid=3000

RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories; \
    apk add --no-cache bash gettext nmap-ncat openssl busybox-extras

# RocketMQ is run with user `rocketmq`, uid = 3000
# If you bind mount a volume from the host or a data container,
# ensure you use the same uid
RUN addgroup --gid ${gid} ${group} \
    && adduser --uid ${uid} -G ${group} ${user} -s /bin/bash -D \
    && cd /home/rocketmq \
    && mkdir logs store \
    && chown -R 3000:3000 logs && chown -R 3000:3000 store

# Rocketmq version
ENV ROCKETMQ_VERSION ${version}
# Rocketmq home
ENV ROCKETMQ_HOME  /home/rocketmq/rocketmq-${ROCKETMQ_VERSION}

WORKDIR  ${ROCKETMQ_HOME}

# Install
RUN set -eux; \
    apk add --virtual .build-deps curl gnupg unzip; \
    curl -L https://mirror.bit.edu.cn/apache/rocketmq/${ROCKETMQ_VERSION}/rocketmq-all-${ROCKETMQ_VERSION}-bin-release.zip -o rocketmq.zip; \
    curl -L https://archive.apache.org/dist/rocketmq/${ROCKETMQ_VERSION}/rocketmq-all-${ROCKETMQ_VERSION}-bin-release.zip.asc -o rocketmq.zip.asc; \
    #https://www.apache.org/dist/rocketmq/KEYS
    curl -L https://www.apache.org/dist/rocketmq/KEYS -o KEYS; \
    \
    gpg --import KEYS; \
    gpg --batch --verify rocketmq.zip.asc rocketmq.zip; \
    unzip rocketmq.zip; \
    mv rocketmq-all*/* . ; \
    rmdir rocketmq-all* ; \
    rm rocketmq.zip rocketmq.zip.asc KEYS; \
    apk del .build-deps ; \
    rm -rf /var/cache/apk/* ; \
    rm -rf /tmp/*

# Copy customized scripts
COPY scripts/ ${ROCKETMQ_HOME}/bin/

# Copy broker config
COPY ./broker.conf ${ROCKETMQ_HOME}/conf/

RUN chown -R ${uid}:${gid} ${ROCKETMQ_HOME}

# Expose namesrv port
EXPOSE 9876

# Override customized scripts for namesrv
RUN mv ${ROCKETMQ_HOME}/bin/runserver-customize.sh ${ROCKETMQ_HOME}/bin/runserver.sh \
    && chmod a+x ${ROCKETMQ_HOME}/bin/runserver.sh \
    && chmod a+x ${ROCKETMQ_HOME}/bin/mqnamesrv

# Expose broker ports
EXPOSE 10909 10911 10912

# Override customized scripts for broker
RUN mv ${ROCKETMQ_HOME}/bin/runbroker-customize.sh ${ROCKETMQ_HOME}/bin/runbroker.sh \
    && chmod a+x ${ROCKETMQ_HOME}/bin/runbroker.sh \
    && chmod a+x ${ROCKETMQ_HOME}/bin/mqbroker

# Export Java options
RUN export JAVA_OPT=" -Duser.home=/opt"

# Add ${JAVA_HOME}/lib/ext as java.ext.dirs
RUN sed -i 's/${JAVA_HOME}\/jre\/lib\/ext/${JAVA_HOME}\/jre\/lib\/ext:${JAVA_HOME}\/lib\/ext/' ${ROCKETMQ_HOME}/bin/tools.sh

USER ${user}

WORKDIR ${ROCKETMQ_HOME}/bin
