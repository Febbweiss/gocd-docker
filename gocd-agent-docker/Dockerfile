FROM docker

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
RUN curl -sL -o go-agent-16.1.0.zip "https://download.go.cd/binaries/16.1.0-2855/generic/go-agent-16.1.0-2855.zip" \
    && unzip go-agent-16.1.0.zip \
    && mv go-agent-16.1.0 go-agent \
    && rm go-agent-16.1.0.zip

RUN rm -rf /tmp/* /var/tmp/* /var/cache/apk/*

RUN git config --global url."https://".insteadOf git://

#VOLUME ["/var/lib/go-agent", "/var/log/go-agent", "/var/go/.ssh"]

COPY autoregister.properties /var/lib/go-agent/config/autoregister.properties

#ENTRYPOINT ["/bin/bash", "-c"]

CMD until curl -s -o /dev/null "http://${GO_SERVER}:${GO_SERVER_PORT}"; do sleep 5; done; \
    if [ -n "$AGENT_KEY" ]; then \
        sed -i -e 's/=.*/=/g' -e "s/key=/key=$AGENT_KEY/" -e "s/resources=/resources=$AGENT_RESOURCES/" /var/lib/go-agent/config/autoregister.properties; \ 
    fi; \
    (/var/lib/go-agent/agent.sh &); while [ ! -f /var/lib/go-agent/go-agent-bootstrapper.log ]; do sleep 1; done; exec tail -F /var/lib/go-agent/*.log

