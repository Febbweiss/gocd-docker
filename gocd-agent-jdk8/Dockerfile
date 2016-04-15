FROM frolvlad/alpine-oraclejdk8

ARG GO_AGENT_VERSION=16.1.0
ARG GO_AGENT_BUILD=2855

RUN apk update \
    && apk add --no-cache \
       libc6-compat \
       curl \
       git \
       subversion \
       bash \
       libstdc++ \
    && ln -fs /bin/bash /bin/sh

WORKDIR /var/lib
RUN curl -sL "http://wwwftp.ciril.fr/pub/apache/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz" | tar -zx
RUN curl -sL -o gradle-2.11-all.zip "https://services.gradle.org/distributions/gradle-2.11-all.zip" \
    && unzip -q gradle-2.11-all.zip \
    && rm gradle-2.11-all.zip \
    && rm -rf gradle-2.11/samples gradle-2.11/docs gradle-2.11/media  gradle-2.11/src

ENV MAVEN_HOME /var/lib/apache-maven-3.3.9
ENV GRADLE_HOME /var/lib/gradle-2.11
ENV PATH $PATH:$MAVEN_HOME/bin:$GRADLE_HOME/bin

# Installing Go agent
RUN curl -sL -o go-agent.zip "https://download.go.cd/binaries/${GO_AGENT_VERSION}-${GO_AGENT_BUILD}/generic/go-agent-${GO_AGENT_VERSION}-${GO_AGENT_BUILD}.zip" \
    && unzip -q go-agent.zip \
    && mv go-agent-${GO_AGENT_VERSION} go-agent \
    && rm go-agent.zip

RUN git config --global url."https://".insteadOf git://

RUN rm -rf /tmp/* /var/tmp/* /var/cache/apk/*

#VOLUME ["/var/lib/go-agent", "/var/log/go-agent", "/var/go/.ssh"]

COPY autoregister.properties /var/lib/go-agent/config/autoregister.properties

#ENTRYPOINT ["/bin/bash", "-c"]

CMD until curl -s -o /dev/null "http://${GO_SERVER}:${GO_SERVER_PORT}"; do sleep 5; done; \
    if [ -n "$AGENT_KEY" ]; then \
        sed -i -e 's/=.*/=/g' -e "s/key=/key=$AGENT_KEY/" -e "s/resources=/resources=$AGENT_RESOURCES/" /var/lib/go-agent/config/autoregister.properties; \
    fi; \
    (/var/lib/go-agent/agent.sh &); while [ ! -f /var/lib/go-agent/go-agent-bootstrapper.log ]; do sleep 1; done; exec tail -F /var/lib/go-agent/*.log
