FROM openshift/jenkins-2-centos7

ENV JAVA_HOME /usr/lib/jvm/jre

# ODS defaults, available to use within pipelines.
ARG ODS_NAMESPACE
ARG ODS_GIT_REF
ARG ODS_IMAGE_TAG
ARG SONAR_EDITION
ARG SONAR_VERSION
ARG APP_DNS

USER root

COPY ./import_certs.sh /usr/local/bin/import_certs.sh
RUN import_certs.sh

# Copy configuration and plugins.
COPY plugins.txt /opt/openshift/configuration/plugins.txt
COPY kube-slave-common.sh /usr/local/bin/kube-slave-common.sh
RUN /usr/local/bin/install-plugins.sh /opt/openshift/configuration/plugins.txt \
    && rm -r /opt/openshift/configuration/jobs/OpenShift* \
    && touch /var/lib/jenkins/configured
COPY configuration/ /opt/openshift/configuration/
COPY ods-run /usr/libexec/s2i/ods-run

RUN chown :0 /etc/pki/java/cacerts && chmod ugo+w /etc/pki/java/cacerts

USER jenkins
ENV JENKINS_JAVA_OVERRIDES="-Dhudson.tasks.MailSender.SEND_TO_UNKNOWN_USERS=true -Dhudson.tasks.MailSender.SEND_TO_USERS_WITHOUT_READ=true"
CMD ["/usr/libexec/s2i/ods-run"]
