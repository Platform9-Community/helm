from jenkins/jenkins:lts
USER root

# Plugins
RUN /bin/jenkins-plugin-cli ws-cleanup workflow-aggregator github greenballs simple-theme-plugin kubernetes docker-workflow kubernetes-cli github-branch-source docker-build-publish matrix-auth

# install Programs
RUN apt-get update && apt-get install -y \
    maven \
    docker \
    gettext \
    sudo \
    curl \
    ruby

RUN curl -sL https://deb.nodesource.com/setup_12.x -o nodesource_setup.sh
RUN bash nodesource_setup.sh
RUN apt-get install -y nodejs

# Install Kubectl
RUN  wget https://storage.googleapis.com/kubernetes-release/release/v1.17.0/bin/linux/amd64/kubectl && chmod +x ./kubectl && mv ./kubectl /usr/local/bin/kubectl
