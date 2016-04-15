FROM docker

ARG GO_AGENT_VERSION=16.1.0
ARG GO_AGENT_BUILD=2855

ENV JAVA_HOME="/usr/lib/jvm/default-jvm"

RUN apk update \
    && apk add --no-cache \
       curl \
       git \
       subversion \
       openjdk7-jre-base \
       bash \
    && ln -fs /bin/bash /bin/sh

# Installing Go agent
WORKDIR /var/lib
RUN curl -sL -o go-agent.zip "https://download.go.cd/binaries/${GO_AGENT_VERSION}-${GO_AGENT_BUILD}/generic/go-agent-${GO_AGENT_VERSION}-${GO_AGENT_BUILD}.zip" \
    && unzip go-agent.zip \
    && mv go-agent-${GO_AGENT_VERSION} go-agent \
    && rm go-agent.zip

RUN rm -rf /tmp/* /var/tmp/* /var/cache/apk/*

RUN git config --global url."https://".insteadOf git://

#VOLUME ["/var/lib/go-agent", "/var/log/go-agent", "/var/go/.ssh"]

COPY autoregister.properties /var/lib/go-agent/config/autoregister.properties

#ENTRYPOINT ["/bin/bash", "-c"]

CMD until curl -s -o /dev/null "http://${GO_SERVER}:${GO_SERVER_PORT}"; do sleep 5; done; \
    if [ -n "$AGENT_KEY" ]; then \
        sed -i -e 's/=.*/=/g' -e "s/key=/key=$AGENT_KEY/" -e "s/resources=/resources=$AGENT_RESOURCES/" /var/lib/go-agent/config/autoregister.properties; \
    fi; \
    (/var/lib/go-agent/agent.sh &); (/usr/local/bin/docker daemon -p /var/run/docker-ssd.pid &);while [ ! -f /var/lib/go-agent/go-agent-bootstrapper.log ]; do sleep 1; done; exec tail -F /var/lib/go-agent/*.log
