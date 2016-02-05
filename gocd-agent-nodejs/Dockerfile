FROM debian

RUN apt-get clean && apt-get update \
    && apt-get install -y \
       curl \
       default-jre-headless \
       git \
       subversion 
RUN apt-get clean && apt-get install -y mongodb
RUN apt-get clean && apt-get install -y python-dev
RUN curl -sL https://deb.nodesource.com/setup_5.x | bash -E - \
    && apt-get update \
    && apt-get install -y \
       build-essential \
       nodejs \
    && apt-get clean \
    && rm -rf /var/lib/apt/listss/*

# Installing Go agent
RUN cd /tmp \
    && curl -sL -o go-agent.deb "https://download.go.cd/gocd-deb/go-agent-15.2.0-2248.deb" \
    && dpkg -i -E go-agent.deb \
    && sed -i -e 's#GO_SERVER=127.0.0.1#GO_SERVER=gocd-server#' /etc/default/go-agent

RUN git config --global url."https://".insteadOf git://

#VOLUME ["/var/lib/go-agent", "/var/log/go-agent", "/var/go/.ssh"]

COPY autoregister.properties /var/lib/go-agent/config/autoregister.properties

CMD until curl -s -o /dev/null "http://${GO_SERVER}:${GO_SERVER_PORT}"; do sleep 5; done; \
    if [ -n "$AGENT_KEY" ]]; then \
        sed -i -e 's/=.*/=/g' -e "s/key=/key=$AGENT_KEY/" -e "s/resources=/resources=$AGENT_RESOURCES/" /var/lib/go-agent/config/autoregister.properties; \ 
    fi; \
    (/usr/share/go-agent/agent.sh &); (service mongodb start); while [ ! -f /var/log/go-agent/go-agent-bootstrapper.log ]; do sleep 1; done; exec tail -F /var/log/go-agent/*

