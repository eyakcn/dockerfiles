FROM jenkins_raw

RUN apt-get -y upgrade && apt-get -y update && apt-get -y autoremove && \
  npm install -g gulp balanced-match grunt-cli bower babel-cli node-gyp cnpm && \
  gem install compass

ENV DOCKER_MACHINE_VERSION v0.6.0-rc1
RUN curl -L "https://github.com/docker/machine/releases/download/$DOCKER_MACHINE_VERSION/docker-machine_linux-amd64" >/usr/local/bin/docker-machine && \
  chmod +x /usr/local/bin/docker-machine

COPY plugins.txt /var/jenkins_home/plugins.txt
RUN /usr/local/bin/plugins.sh /var/jenkins_home/plugins.txt

RUN git config --global user.email "dev@xxxx.com" && \
  git config --global user.name "Jenkins"

RUN ln -s -f /lib/x86_64-linux-gnu/libdevmapper.so.1.02.1 /lib/x86_64-linux-gnu/libdevmapper.so.1.02

EXPOSE 8080

CMD [""]
