# https://github.com/keaz/jenkins/tree/master/master
# https://www.youtube.com/watch?v=768SxaFkvQ4

#FROM python:3.8.6-buster AS builder

## docker-compose requires pyinstaller 3.6 (check github.com/docker/compose/requirements-build.txt)
#ARG VERSION=1.27.4
#ARG PYINSTALLER_VER=3.6

#ENV LANG C.UTF-8

#RUN mkdir -p /build/pyinstallerbootloader && \
#		# Compile the pyinstaller "bootloader"
#		# https://pyinstaller.readthedocs.io/en/stable/bootloader-building.html
#		cd /build/pyinstallerbootloader && \
#		curl -sSL https://github.com/pyinstaller/pyinstaller/releases/download/v$PYINSTALLER_VER/PyInstaller-$PYINSTALLER_VER.tar.gz | tar xz --strip 1 && \
#		cd bootloader && python3 ./waf all && \
#		# Clone docker-compose
#		mkdir -p /build/dockercompose && \
#		cd /build/dockercompose && \
#		git clone https://github.com/docker/compose.git . && \
#		git checkout $VERSION && \
#		# ./script/build/write-git-sha && \
#		git rev-parse --short HEAD > compose/GITSHA && \
#		# Run the build steps (taken from github.com/docker/compose/script/build/linux-entrypoint)
#		mkdir ./dist && \
#		pip install -q -r requirements.txt -r requirements-build.txt && \
#		pyinstaller docker-compose.spec && \
#		mv dist/docker-compose /usr/bin/docker-compose


FROM adoptopenjdk/openjdk11:alpine-slim

#COPY --from=builder /usr/bin/docker-compose /usr/bin/docker-compose

RUN apk add --no-cache \
		bash \
		coreutils \
		curl \
		git \
		git-lfs \
		openssh-client \
		tini \
		ttf-dejavu \
		tzdata \
		unzip \
		\
#		maven docker 
	\
		# Install kubectl
		&& curl -Lo /usr/bin/kubectl "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl" \
		&& chmod +x /usr/bin/kubectl

ARG user=jenkins
ARG group=jenkins
ARG uid=1000
ARG gid=1000
ARG http_port=8080
ARG agent_port=50000
ARG JENKINS_HOME=/var/jenkins_home
ARG REF=/usr/share/jenkins/ref

ENV JENKINS_HOME $JENKINS_HOME
ENV JENKINS_SLAVE_AGENT_PORT ${agent_port}
ENV REF $REF

# Jenkins is run with user `jenkins`, uid = 1000
# If you bind mount a volume from the host or a data container,
# ensure you use the same uid
RUN mkdir -p $JENKINS_HOME \
		&& chown ${uid}:${gid} $JENKINS_HOME \
		&& addgroup -g ${gid} ${group} \
		&& adduser -h "$JENKINS_HOME" -u ${uid} -G ${group} -s /bin/bash -D ${user}

# Jenkins home directory is a volume, so configuration and build history
# can be persisted and survive image upgrades
VOLUME $JENKINS_HOME

# $REF (defaults to `/usr/share/jenkins/ref/`) contains all reference configuration we want
# to set on a fresh new installation. Use it to bundle additional plugins
# or config file with your custom jenkins Docker image.
RUN mkdir -p ${REF}/init.groovy.d

# jenkins version being bundled in this docker image
ARG JENKINS_VERSION
ENV JENKINS_VERSION ${JENKINS_VERSION:-2.330}

# jenkins.war checksum, download will be validated using it
# https://updates.jenkins-ci.org/download/war/
ARG JENKINS_SHA=7cf5fd643a23aa55c9c58452d9ee0eac9efc7ba89f172ab0e669d31f910731a1

# Can be used to customize where jenkins.war get downloaded from
ARG JENKINS_URL=https://repo.jenkins-ci.org/public/org/jenkins-ci/main/jenkins-war/${JENKINS_VERSION}/jenkins-war-${JENKINS_VERSION}.war

# could use ADD but this one does not check Last-Modified header neither does it allow to control checksum
# see https://github.com/docker/docker/issues/8331
RUN curl -fsSL ${JENKINS_URL} -o /usr/share/jenkins/jenkins.war \
  && echo "${JENKINS_SHA}  /usr/share/jenkins/jenkins.war" | sha256sum -c -

ENV JENKINS_UC https://updates.jenkins.io
ENV JENKINS_UC_EXPERIMENTAL=https://updates.jenkins.io/experimental
ENV JENKINS_INCREMENTALS_REPO_MIRROR=https://repo.jenkins-ci.org/incrementals
RUN chown -R ${user} "$JENKINS_HOME" "$REF"

# for main web interface:
EXPOSE ${http_port}

# will be used by attached slave agents:
EXPOSE ${agent_port}

ENV COPY_REFERENCE_FILE_LOG $JENKINS_HOME/copy_reference_file.log

USER ${user}

COPY jenkins-support /usr/local/bin/jenkins-support
COPY jenkins.sh /usr/local/bin/jenkins.sh
COPY tini-shim.sh /bin/tini
ENTRYPOINT ["/sbin/tini", "--", "/usr/local/bin/jenkins.sh"]

# from a derived Dockerfile, can use `RUN plugins.sh active.txt` to setup $REF/plugins from a support bundle
COPY plugins.sh /usr/local/bin/plugins.sh
COPY install-plugins.sh /usr/local/bin/install-plugins.sh

COPY plugins.txt /usr/share/jenkins/ref/plugins.txt
RUN /usr/local/bin/install-plugins.sh < /usr/share/jenkins/ref/plugins.txt

USER root
## Install plugins for master jenkins
#RUN \
#	/usr/local/bin/install-plugins.sh \
#			# Distributed Builds plugins
#			ssh-slaves \
#			#Install GIT
#			git \
#			# Artifacts
#			htmlpublisher \
#			# UI
#			greenballs \
#			simple-theme-plugin \
#			# Scaling
#			kubernetes

