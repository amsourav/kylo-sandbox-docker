FROM centos:latest

ENV BRANCH=master
ENV MAVEN_PROJECTS_LIST="!:kylo-install-tar,!:kylo-docs,!:kylo-samples,!:kylo-samples-plugins,!:kylo-install-rpm,!:kylo-install-debian,!:example-auth-custom,!:example-ui-get-file-processor-template,!:example-ui-feed-stepper-plugin,!:example-module-services,!:example-module-ui,!:kylo-provenance-samples,!:kylo-spark-provenance-app,!:kylo-release"

RUN yum install -yy wget git which java-1.8.0-openjdk-devel bzip2 rpm-build curl sudo unzip mariadb

# Install NodeJS and Yarn
RUN curl -sLk https://rpm.nodesource.com/setup_10.x | bash - && yum install nodejs -yy
RUN curl -sLk https://dl.yarnpkg.com/rpm/yarn.repo -o /etc/yum.repos.d/yarn.repo && yum install yarn -yy

# Install wkhtmltopdf (https://gist.github.com/paulsturgess/cfe1a59c7c03f1504c879d45787699f5)
RUN yum install -yy libpng libjpeg openssl icu libX11 libXext libXrender xorg-x11-fonts-Type1 xorg-x11-fonts-75dpi
RUN wget --no-check-certificate https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.5/wkhtmltox-0.12.5-1.centos7.x86_64.rpm
RUN rpm -Uvh wkhtmltox-0.12.5-1.centos7.x86_64.rpm

# Install Maven
RUN mkdir -p /usr/local/src && cd /usr/local/src && wget --no-check-certificate http://apache.crihan.fr/dist/maven/maven-3/3.5.4/binaries/apache-maven-3.5.4-bin.tar.gz
RUN cd /usr/local/src && tar -xf apache-maven-3.5.4-bin.tar.gz && rm -f apache-maven-3.5.4-bin.tar.gz && mv apache-maven-3.5.4/ apache-maven/

# Checkout kylo
RUN mkdir -p /usr/local/src/ && cd /usr/local/src/ && git clone --branch $BRANCH https://github.com/amsourav/kylo.git

# Copy kylo TAR
RUN cd /usr/local/src/kylo/ && /usr/local/src/apache-maven/bin/mvn package -B -pl "$MAVEN_PROJECTS_LIST" -DskipTests=true -Dlicense.skipCheckLicense=true -Dorg.slf4j.simpleLogger.log.org.apache.maven.cli.transfer.Slf4jMavenTransferListener=warn && mkdir -p /opt/kylo && mv /usr/local/src/kylo/install/install-tar/target/kylo-*-dependencies.tar.gz /opt/kylo/ && cd /opt/kylo/ && tar zxvf kylo-*-dependencies.tar.gz && rm -f kylo-*-dependencies.tar.gz && rm -rf /toot/.m2 /usr/local/src/apache-maven /usr/local/src/apache-maven /usr/local/src/kylo

# Install Kylo
RUN useradd -r -G users kylo

RUN /opt/kylo/setup/install/post-install.sh