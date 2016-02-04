FROM debian

RUN apt-get clean && apt-get update \
    && apt-get install -y curl
RUN apt-get clean && apt-get install -y maven
RUN apt-get clean && apt-get install -y gradle \
    && apt-get clean \
    && rm -rf /var/lib/apt/listss/*

WORKDIR /opt
#RUN curl -sL -o - --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" -q "http://download.oracle.com/otn-pub/java/jdk/8u66-b17/jdk-8u66-linux-x64.tar.gz" | tar xzf - 
ADD binaries/jdk-8u66-linux-x64.tar.gz /opt/.
RUN update-alternatives --install /usr/bin/java java /opt/jdk1.8.0_66/bin/java 120 \
    && update-alternatives --install /usr/bin/keytool keytool /opt/jdk1.8.0_66/bin/keytool 120\
    && update-alternatives --install /usr/bin/rmiregistry rmiregistry /opt/jdk1.8.0_66/bin/rmiregistry 120\
    && update-alternatives --install /usr/bin/javac javac /opt/jdk1.8.0_66/bin/javac 120 \
	--slave /usr/bin/jar  jar  /opt/jdk1.8.0_66/bin/jar \
	--slave /usr/bin/rmic rmic /opt/jdk1.8.0_66/bin/rmic\
    && update-alternatives --set java /opt/jdk1.8.0_66/bin/java\
    && update-alternatives --set keytool /opt/jdk1.8.0_66/bin/keytool \
    && update-alternatives --set rmiregistry /opt/jdk1.8.0_66/bin/rmiregistry \
    && echo "export JAVA_HOME=/opt/jdk1.8.0_66" >> /.profile

# Installing Go agent
RUN cd /tmp \
    && curl -sL -o go-agent.deb "https://download.go.cd/gocd-deb/go-agent-15.2.0-2248.deb" \
    && dpkg -i -E go-agent.deb \
    && sed -i -e 's#GO_SERVER=127.0.0.1#GO_SERVER=gocd-server#' /etc/default/go-agent \
    && sed -i -e 's#JAVA_HOME="/usr/lib/jvm/java-7-openjdk-amd64/jre"#JAVA_HOME="/opt/jdk1.8.0_66/jre"#' /etc/default/go-agent

#VOLUME ["/var/lib/go-agent", "/var/log/go-agent", "/var/go/.ssh"]

COPY autoregister.properties /var/lib/go-agent/config/autoregister.properties

CMD until curl -s -o /dev/null "http://${GO_SERVER}:${GO_SERVER_PORT}"; do sleep 5; done; \
    if [ -n "$AGENT_KEY" ]]; then \
        sed -i -e 's/=.*/=/g' -e "s/key=/key=$AGENT_KEY/" -e "s/resources=/resources=$AGENT_RESOURCES/" /var/lib/go-agent/config/autoregister.properties; \ 
    fi; \
    (/usr/share/go-agent/agent.sh &); while [ ! -f /var/log/go-agent/go-agent-bootstrapper.log ]; do sleep 1; done; exec tail -F /var/log/go-agent/*

