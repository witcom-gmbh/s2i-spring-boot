ARG BASE_IMAGE=openjdk:17-alpine3.14
FROM $BASE_IMAGE

MAINTAINER Carsten Buchberger c.buchberger@witcom.de

LABEL org.opencontainers.image.title="Base-Container for S2I-SPRING-BOOT" \
      org.opencontainers.image.source="https://github.com/witcom-gmbh/s2i-spring-boot" \
      io.openshift.s2i.scripts-url="image:///usr/libexec/s2i" \
      io.s2i.scripts-url="image:///usr/libexec/s2i"

# Install required util packages.
RUN apk update && apk add ca-certificates java-cacerts tar unzip sudo curl bash git libatomic && rm -rf /var/cache/apk/* 

# Create required directories.
RUN mkdir -p /opt/openshift && mkdir -p /opt/app-root/src
ENV HOME=/opt/app-root/src

# Install Maven 3.6.3
ENV MAVEN_VERSION 3.6.3
RUN (curl -fSL https://artfiles.org/apache.org/maven/maven-3/$MAVEN_VERSION/binaries/apache-maven-$MAVEN_VERSION-bin.tar.gz | \
    tar -zx -C /usr/local) && \
    mv /usr/local/apache-maven-$MAVEN_VERSION /usr/local/maven && \
    ln -sf /usr/local/maven/bin/mvn /usr/local/bin/mvn && \
    mkdir -p $HOME/.m2 && chmod -R a+rwX $HOME/.m2

# Install Gradle 4.4
ENV GRADLE_VERSION 4.4
RUN curl -fSL https://services.gradle.org/distributions/gradle-$GRADLE_VERSION-bin.zip -o /tmp/gradle-$GRADLE_VERSION-bin.zip && \
    unzip /tmp/gradle-$GRADLE_VERSION-bin.zip -d /usr/local/ && \
    rm /tmp/gradle-$GRADLE_VERSION-bin.zip && \
    mv /usr/local/gradle-$GRADLE_VERSION /usr/local/gradle && \
    ln -sf /usr/local/gradle/bin/gradle /usr/local/bin/gradle && \
    mkdir -p $HOME/.gradle && chmod -R a+rwX $HOME/.gradle

# Set the location of the mvn and gradle bin directories on search path
ENV PATH=/usr/local/bin/mvn:/usr/local/bin/gradle:$PATH

# Set the default build type to 'Maven'
ENV BUILD_TYPE=Maven

# Drop the root user and make the content of /opt/openshift owned by user 1001
RUN chown -R 1001:1001 /opt/openshift /opt/app-root/src

# Change perms on target/deploy directory to 777
RUN chmod -R 777 /opt/openshift /opt/app-root/src

# Copy the S2I scripts to /usr/libexec/s2i, since openshift/base-centos7 image
# sets io.openshift.s2i.scripts-url label that way.
COPY ./s2i/bin/ /usr/libexec/s2i
RUN chmod -R 777 /usr/libexec/s2i

# This default user is created in the openshift/base-centos7 image
USER 1001

# Set the default port for applications built using this image
EXPOSE 8080
WORKDIR /opt/app-root/src
# Set the default CMD for the image
CMD ["/usr/libexec/s2i/usage"]