FROM node

ARG GO_AGENT_VERSION=16.1.0
ARG GO_AGENT_BUILD=2855

RUN apt-get clean && apt-get update \
    && apt-get install -y \
       curl \
       git \
       subversion \
       unzip \
    && apt-get update \
    && apt-get install -y \
       build-essential \
       nodejs

RUN apt-get clean && apt-get install -y mongodb
RUN apt-get clean \
    && rm -rf /var/lib/apt/listss/* /tmp/* /var/tmp/*

RUN cd /var/lib \
    && curl -sL "http://wwwftp.ciril.fr/pub/apache/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz" | tar -zx \
    && curl -sL -o gradle-2.11-all.zip "https://services.gradle.org/distributions/gradle-2.11-all.zip" \
    && unzip -q gradle-2.11-all.zip \
    && rm gradle-2.11-all.zip \
    && rm -rf gradle-2.11/samples gradle-2.11/docs gradle-2.11/media  gradle-2.11/src

ENV MAVEN_HOME /var/lib/apache-maven-3.3.9
ENV GRADLE_HOME /var/lib/gradle-2.11
ENV PATH $PATH:$MAVEN_HOME/bin:$GRADLE_HOME/bin

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

# Installing CloudFoundry
RUN cd /tmp \
    && curl -sL -o cf-cli-installer_6.15.0_x86-64.deb "https://cli.run.pivotal.io/stable?release=debian64&version=6.15.0&source=github-rel" \
    && dpkg -i -E cf-cli-installer_6.15.0_x86-64.deb \
    && cf add-plugin-repo bluemix-cf-staging http://plugins.ng.bluemix.net >/dev/null 2>&1

# Installing Go agent
RUN cd /tmp \
    && curl -sL -o go-agent.deb "https://download.go.cd/binaries/${GO_AGENT_VERSION}-${GO_AGENT_BUILD}/deb/go-agent-${GO_AGENT_VERSION}-${GO_AGENT_BUILD}.deb" \
    && dpkg -i --ignore-depends=java7-runtime-headless -E go-agent.deb \
    && sed -i -e 's#GO_SERVER=127.0.0.1#GO_SERVER=gocd-server#' /etc/default/go-agent \
    && sed -i -e 's#JAVA_HOME="/usr/lib/jvm/java-7-openjdk-amd64/jre"#JAVA_HOME="/opt/jdk1.8.0_66/jre"#' /etc/default/go-agent

RUN git config --global url."https://".insteadOf git://

#VOLUME ["/var/lib/go-agent", "/var/log/go-agent", "/var/go/.ssh"]

COPY autoregister.properties /var/lib/go-agent/config/autoregister.properties

CMD until curl -s -o /dev/null "http://${GO_SERVER}:${GO_SERVER_PORT}"; do sleep 5; done; \
    if [ -n "$AGENT_KEY" ]]; then \
        sed -i -e 's/=.*/=/g' -e "s/key=/key=$AGENT_KEY/" -e "s/resources=/resources=$AGENT_RESOURCES/" /var/lib/go-agent/config/autoregister.properties; \
    fi; \
    (/usr/share/go-agent/agent.sh &); (service mongodb start); while [ ! -f /var/log/go-agent/go-agent-bootstrapper.log ]; do sleep 1; done; exec tail -F /var/log/go-agent/*
