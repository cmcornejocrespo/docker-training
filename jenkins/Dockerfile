FROM jenkins/jenkins:lts

LABEL mantainer="Carlos Cornejo - cmcornejo@atsistemas.com"
LABEL version="1.0"
LABEL description="HP Dockerfile example"

ENV JAVA_OPTS="-Djenkins.install.runSetupWizard=false"

COPY basic-security.groovy /usr/share/jenkins/ref/init.groovy.d/basic-security.groovy
COPY plugins.txt /usr/share/jenkins/ref/plugins.txt
RUN /usr/local/bin/install-plugins.sh < /usr/share/jenkins/ref/plugins.txt