FROM openshift/jenkins-slave-base-centos7

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ENV SONAR_SCANNER_VERSION=3.1.0.1141 \
    CNES_REPORT_VERSION=3.1.0 \
    TAILOR_VERSION=1.1.4 \
    GIT_LFS_VERSION=2.6.1 \
    SKOPEO_VERSION=0.1.37-3 \
    OSTREE_VERSION=2018.5-1

ARG APP_DNS
ARG SNYK_DISTRIBUTION_URL

RUN yum -y install \
    openssl \
    && yum clean all \
    && rm -rf /var/cache/yum/*

ENV JAVA_HOME=/usr/lib/jvm/jre

RUN yum -y install java-1.8.0-openjdk-devel.x86_64 \
    && yum clean all \
    && rm -rf /var/cache/yum/*

COPY ./import_certs.sh /usr/local/bin/import_certs.sh
RUN import_certs.sh

# Install Sonar Scanner.
RUN cd /tmp \
    && curl -LOv https://repo1.maven.org/maven2/org/sonarsource/scanner/cli/sonar-scanner-cli/${SONAR_SCANNER_VERSION}/sonar-scanner-cli-${SONAR_SCANNER_VERSION}.zip \
    && unzip sonar-scanner-cli-${SONAR_SCANNER_VERSION}.zip \
    && mv sonar-scanner-${SONAR_SCANNER_VERSION} /usr/local/sonar-scanner-cli \
    && rm -rf sonar-scanner-cli-${SONAR_SCANNER_VERSION}.zip \
    && /usr/local/sonar-scanner-cli/bin/sonar-scanner --version
ENV PATH=/usr/local/sonar-scanner-cli/bin:$PATH

# Add sq cnes report jar.
RUN cd /tmp \
    && curl -Lv https://github.com/lequal/sonar-cnes-report/releases/download/${CNES_REPORT_VERSION}/sonar-cnes-report-${CNES_REPORT_VERSION}.jar -o cnesreport.jar \
    && mkdir /usr/local/cnes \
    && mv cnesreport.jar /usr/local/cnes/cnesreport.jar \
    && chmod 777 /usr/local/cnes/cnesreport.jar

# Install Tailor.
RUN cd /tmp \
	&& curl -LOv https://github.com/opendevstack/tailor/releases/download/v${TAILOR_VERSION}/tailor-linux-amd64 \
	&& mv tailor-linux-amd64 /usr/local/bin/tailor \
	&& chmod a+x /usr/local/bin/tailor \
	&& tailor version

# Install GIT-LFS extension https://git-lfs.github.com/.
RUN cd /tmp \
    && mkdir -p /tmp/git-lfs \
    && curl -LOv https://github.com/git-lfs/git-lfs/releases/download/v${GIT_LFS_VERSION}/git-lfs-linux-amd64-v${GIT_LFS_VERSION}.tar.gz \
    && tar -zxvf git-lfs-linux-amd64-v${GIT_LFS_VERSION}.tar.gz -C /tmp/git-lfs \
    && bash /tmp/git-lfs/install.sh \
    && git lfs version \
    && rm -rf /tmp/git-lfs*

# Optionally install snyk.
RUN if [ -z $SNYK_DISTRIBUTION_URL ] ; then echo 'Skipping snyk installation!' ; else echo 'Installing snyk... getting binary from' $SNYK_DISTRIBUTION_URL \
    && curl -Lv $SNYK_DISTRIBUTION_URL --output snyk \
    && mv snyk /usr/local/bin \
    && chmod +rwx /usr/local/bin/snyk \
    && mkdir -p $HOME/.config/configstore/ \
    && chmod -R g+rw $HOME/.config/configstore/ \
    && echo 'Snyk CLI version:' \
    && snyk --version \
    && echo 'Snyk installation completed!'; \
    fi

# Set java proxy var.
COPY set_java_proxy.sh /tmp/set_java_proxy.sh
RUN cp /usr/local/bin/run-jnlp-client /usr/local/bin/ocp-run-jnlp-client
COPY ods-run-jnlp-client /usr/local/bin/run-jnlp-client
RUN chmod +x /usr/local/bin/run-jnlp-client \
    && chmod +x /tmp/set_java_proxy.sh \
    && . /tmp/set_java_proxy.sh && echo $JAVA_OPTS

# Add skopeo.
RUN yum install -y \
    http://mirror.centos.org/centos/7/extras/x86_64/Packages/containers-common-${SKOPEO_VERSION}.el7.centos.x86_64.rpm \
    http://mirror.centos.org/centos/7/extras/x86_64/Packages/ostree-${OSTREE_VERSION}.el7.x86_64.rpm \
    http://mirror.centos.org/centos/7/extras/x86_64/Packages/skopeo-${SKOPEO_VERSION}.el7.centos.x86_64.rpm \
    && yum clean all \
    && rm -rf /var/cache/yum/* \
    && skopeo --version

# Fix permissions.
RUN mkdir -p /home/jenkins/.config && chmod g+w /home/jenkins/.config \
    && mkdir -p /home/jenkins/.cache && chmod g+w /home/jenkins/.cache

RUN chmod g+w $JAVA_HOME/lib/security/cacerts
