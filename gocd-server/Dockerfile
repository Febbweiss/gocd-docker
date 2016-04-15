FROM debian

ARG GO_SERVER_VERSION=16.1.0
ARG GO_SERVER_BUILD=2855

RUN apt-get clean && apt-get update \
    && apt-get install -y \
       curl \
       default-jre-headless
RUN apt-get clean && apt-get install -y \
       git \
       subversion
RUN apt-get clean && apt-get install -y maven
RUN apt-get clean && apt-get install -y gradle \
    && apt-get clean

# Setting Java8 as default Java
WORKDIR /opt
RUN curl -sL -o - --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" -q "http://download.oracle.com/otn-pub/java/jdk/8u66-b17/jdk-8u66-linux-x64.tar.gz" | tar xzf - && \
    update-alternatives --install /usr/bin/java java /opt/jdk1.8.0_66/bin/java 120 \
    && update-alternatives --install /usr/bin/keytool keytool /opt/jdk1.8.0_66/bin/keytool 120\
    && update-alternatives --install /usr/bin/rmiregistry rmiregistry /opt/jdk1.8.0_66/bin/rmiregistry 120\
    && update-alternatives --install /usr/bin/javac javac /opt/jdk1.8.0_66/bin/javac 120 \
	--slave /usr/bin/jar  jar  /opt/jdk1.8.0_66/bin/jar \
	--slave /usr/bin/rmic rmic /opt/jdk1.8.0_66/bin/rmic\
    && update-alternatives --set java /opt/jdk1.8.0_66/bin/java\
    && update-alternatives --set keytool /opt/jdk1.8.0_66/bin/keytool \
    && update-alternatives --set rmiregistry /opt/jdk1.8.0_66/bin/rmiregistry \
    && echo "export JAVA_HOME=/opt/jdk1.8.0_66" >> /.profile

# Installing Go server
RUN cd /tmp \
    && curl -sL -o go-server.deb "https://download.go.cd/binaries/${GO_SERVER_VERSION}-${GO_SERVER_BUILD}/deb/go-server-${GO_SERVER_VERSION}-${GO_SERVER_BUILD}.deb" \
    && dpkg -i go-server.deb \
    && rm -f go-server.deb \
    && sed -i -e "s/DAEMON=Y/DAEMON=N/" /etc/default/go-server \
    && sed -i -e 's#JAVA_HOME="/usr/lib/jvm/java-7-openjdk-amd64/jre"#JAVA_HOME="/opt/jdk1.8.0_66/jre"#' /etc/default/go-server

RUN git config --global url."https://".insteadOf git://

RUN rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

VOLUME ["/var/lib/go-server", "/var/log/go-server", "/etc/go", "/var/go/.ssh"]

# Building CloudFoundry plugin
RUN cd /tmp \
    && curl -sL -o springer-gocd-cloudfoundry-plugin-2.0.1.jar "https://github.com/Sounie/springer-gocd-cloudfoundry-plugin/releases/download/v2.0.0/springer-gocd-cloudfoundry-plugin-2.0.0.jar"

# expose ports
EXPOSE 8153 8154

ENV SERVER_MEM 512m
ENV SERVER_MAX_MEM 1024m
ENV AGENT_KEY ""

CMD if [ ! -f /var/lib/go-server/plugins/external/springer-gocd-cloudfoundry-plugin-2.0.1.jar ]]; then \
        mkdir -p /var/lib/go-server/plugins/external; \
        cp /tmp/springer-gocd-cloudfoundry-plugin-2.0.1.jar /var/lib/go-server/plugins/external/springer-gocd-cloudfoundry-plugin-2.0.1.jar; \
     fi; \
     (/bin/su - $go -c "/usr/share/go-server/server.sh &"); \
     until curl -s -o /dev/null 'http://localhost:8153'; \
     do \
        sleep 1; \
     done; \
     if [ -n "$AGENT_KEY" ]]; then \
        sed -i -e 's/agentAutoRegisterKey="[^"]*" *//' -e 's#\(<server\)\(.*artifactsdir.*\)#\1 agentAutoRegisterKey="'$AGENT_KEY'"\2#' /etc/go/cruise-config.xml; \
     fi; \
     exec tail -F /var/log/go-server/*
