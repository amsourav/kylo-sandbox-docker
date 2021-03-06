FROM centos:latest as builder

ENV BRANCH=master
ENV MAVEN_PROJECTS_LIST="!:kylo-docs,!:kylo-samples,!:kylo-samples-plugins,!:kylo-install-rpm,!:kylo-install-debian,!:example-auth-custom,!:example-ui-get-file-processor-template,!:example-ui-feed-stepper-plugin,!:example-module-services,!:example-module-ui,!:kylo-provenance-samples,!:kylo-spark-provenance-app,!:kylo-release"

RUN yum install -yy wget git which java-1.8.0-openjdk-devel bzip2 rpm-build curl sudo unzip mariadb

# Install Maven
RUN mkdir -p /usr/local/src && cd /usr/local/src && wget --no-check-certificate http://apache.crihan.fr/dist/maven/maven-3/3.5.4/binaries/apache-maven-3.5.4-bin.tar.gz
RUN cd /usr/local/src && tar -xf apache-maven-3.5.4-bin.tar.gz && rm -f apache-maven-3.5.4-bin.tar.gz && mv apache-maven-3.5.4/ apache-maven/

# Checkout kylo
RUN mkdir -p /usr/local/src/ && cd /usr/local/src/ && git clone --branch $BRANCH https://github.com/amsourav/kylo.git

# Install && Copy kylo TAR
RUN cd /usr/local/src/kylo/ && /usr/local/src/apache-maven/bin/mvn package -B -pl "$MAVEN_PROJECTS_LIST" -DskipTests=true -Dlicense.skipCheckLicense=true -Dorg.slf4j.simpleLogger.log.org.apache.maven.cli.transfer.Slf4jMavenTransferListener=warn && mkdir -p /opt/kylo && mv /usr/local/src/kylo/install/install-tar/target/kylo-*-dependencies.tar.gz /opt/kylo/ && cd /opt/kylo/ && tar zxvf kylo-*-dependencies.tar.gz && rm -f kylo-*-dependencies.tar.gz && rm -rf /root/.m2 /usr/local/src/apache-maven /usr/local/src/apache-maven /usr/local/src/kylo

RUN useradd -r -G users kylo

RUN /opt/kylo/setup/install/post-install.sh

# Create additional shells required for Docker
RUN sed -E 's/(^java .*) > .+/\1/' /opt/kylo/kylo-ui/bin/run-kylo-ui.sh > /opt/kylo/kylo-ui/bin/run-kylo-ui-interactive.sh && chmod +x /opt/kylo/kylo-ui/bin/run-kylo-ui-interactive.sh
RUN sed -i 's/file/console/g' /opt/kylo/kylo-ui/conf/log4j.properties && sed -i 's/DailyRollingFileAppender/ConsoleAppender/g' /opt/kylo/kylo-ui/conf/log4j.properties
RUN sed 's/localhost:8400/$1:$2/g' /opt/kylo/setup/data/install-templates-locally.sh > /opt/kylo/setup/data/install-templates.sh && chmod +x /opt/kylo/setup/data/install-templates.sh

RUN sed -E 's/(^java .*) > .+/\1/' /opt/kylo/kylo-services/bin/run-kylo-services.sh > /opt/kylo/kylo-services/bin/run-kylo-services-interactive.sh && chmod +x /opt/kylo/kylo-services/bin/run-kylo-services-interactive.sh
RUN sed -i 's/file/console/g' /opt/kylo/kylo-services/conf/log4j.properties && sed -i 's/DailyRollingFileAppender/ConsoleAppender/g' /opt/kylo/kylo-services/conf/log4j.properties

# Enable plugins
RUN cp /opt/kylo/setup/plugins/kylo-service-monitor-ambari-*.jar /opt/kylo/kylo-services/plugin/
RUN cp /opt/kylo/setup/plugins/kylo-service-monitor-activemq-*.jar /opt/kylo/kylo-services/plugin/
RUN cp /opt/kylo/setup/plugins/kylo-service-monitor-kylo-cluster-*.jar /opt/kylo/kylo-services/plugin/

# Install spark
RUN mkdir -p /opt/spark
RUN wget --no-check-certificate https://archive.apache.org/dist/spark/spark-2.3.1/spark-2.3.1-bin-hadoop2.7.tgz
RUN tar zxvf spark-2.3.1-bin-hadoop2.7.tgz -C /opt/spark && rm -f spark-2.3.1-bin-hadoop2.7.tgz
RUN ln -s /opt/spark/spark-2.3.1-bin-hadoop2.7 /opt/spark/current

# Configure Nifi
RUN useradd -r nifi
RUN /opt/kylo/setup/nifi/install-nifi.sh 1.6.0 /opt/nifi nifi nifi
RUN sed -i 's/SPARK_SUBMIT=.*/SPARK_SUBMIT=\/opt\/spark\/current\/bin\/spark-submit/' /opt/kylo/setup/nifi/create-symbolic-links.sh
RUN /opt/kylo/setup/nifi/install-kylo-components.sh /opt/nifi /opt/kylo/ nifi nifi

# Install wait-for-it util
RUN cd /opt && git clone https://github.com/vishnubob/wait-for-it.git && ln -s /opt/wait-for-it/wait-for-it.sh /usr/sbin

## Set entrypoint
WORKDIR /opt/kylo
COPY entrypoint*.sh /
RUN chmod +x /entrypoint*.sh