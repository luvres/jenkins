# https://github.com/keaz/jenkins/tree/master/master
# https://www.youtube.com/watch?v=768SxaFkvQ4
FROM izone/jenkins

RUN /usr/local/bin/install-plugins.sh \
			# Distributed Builds plugins
			ssh-slaves \
			#Install GIT
			git \
			# Artifacts
			htmlpublisher \
			# UI
			greenballs \
			simple-theme-plugin \
			# Scaling
			kubernetes

