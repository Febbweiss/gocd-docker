FROM frolvlad/alpine-oraclejdk8

RUN apk update \
    && apk add --no-cache \
       libc6-compat \
       curl \
       git \
       subversion \
       bash \
    && ln -fs /bin/bash /bin/sh

WORKDIR /var/lib
RUN curl -sL "http://wwwftp.ciril.fr/pub/apache/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz" | tar -zx
RUN curl -sL -o gradle-2.11-all.zip "https://services.gradle.org/distributions/gradle-2.11-all.zip" \
    && unzip gradle-2.11-all.zip \
    && rm gradle-2.11-all.zip \
    && rm -rf gradle-2.11/samples gradle-2.11/docs gradle-2.11/media  gradle-2.11/src

#RUN curl -sL -o - --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" -q "http://download.oracle.com/otn-pub/java/jdk/8u66-b17/jdk-8u66-linux-x64.tar.gz" | tar xzf - 
#ADD binaries/jdk-8u66-linux-x64.tar.gz /opt/.
#RUN update-alternatives --install /usr/bin/java java /opt/jdk1.8.0_66/bin/java 120 \
#    && update-alternatives --install /usr/bin/keytool keytool /opt/jdk1.8.0_66/bin/keytool 120\
#    && update-alternatives --install /usr/bin/rmiregistry rmiregistry /opt/jdk1.8.0_66/bin/rmiregistry 120\
#    && update-alternatives --install /usr/bin/javac javac /opt/jdk1.8.0_66/bin/javac 120 \
#	--slave /usr/bin/jar  jar  /opt/jdk1.8.0_66/bin/jar \
#	--slave /usr/bin/rmic rmic /opt/jdk1.8.0_66/bin/rmic\
#    && update-alternatives --set java /opt/jdk1.8.0_66/bin/java\
#    && update-alternatives --set keytool /opt/jdk1.8.0_66/bin/keytool \
#    && update-alternatives --set rmiregistry /opt/jdk1.8.0_66/bin/rmiregistry 

ENV MAVEN_HOME /var/lib/apache-maven-3.3.9
ENV GRADLE_HOME /var/lib/gradle-2.11
ENV PATH $PATH:$MAVEN_HOME/bin:$GRADLE_HOME/bin

# Installing Go agent
RUN curl -sL -o go-agent-16.1.0.zip "https://download.go.cd/binaries/16.1.0-2855/generic/go-agent-16.1.0-2855.zip" \
    && unzip go-agent-16.1.0.zip \
    && mv go-agent-16.1.0 go-agent \
    && rm go-agent-16.1.0.zip

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

