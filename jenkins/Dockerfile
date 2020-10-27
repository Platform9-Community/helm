from jenkins/jenkins:2.249.2-lts
USER root

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

# Install Kubectl compatible with PMK 4.4 and 4.5.x
RUN  wget https://storage.googleapis.com/kubernetes-release/release/v1.17.0/bin/linux/amd64/kubectl && chmod +x ./kubectl && mv ./kubectl /usr/local/bin/kubectl