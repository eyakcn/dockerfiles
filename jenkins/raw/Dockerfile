FROM ubuntu:14.04

ENV LANGUAGE en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LC_ALL en_US.UTF-8

RUN apt-get update -y && \
  apt-get install -y wget curl zip git nano vim make gcc g++ autoconf zlib1g python-software-properties && \
  rm -rf /var/lib/apt/lists/* && \
  apt-get clean all && \
  locale-gen en_US.UTF-8 && \
  dpkg-reconfigure locales && \
  locale && \
  git config --global push.default simple

# this is a non-interactive automated build - avoid some warning messages
ENV DEBIAN_FRONTEND noninteractive

ENV JENKINS_HOME /var/jenkins_home
VOLUME /var/jenkins_home

ENV JENKINS_UC https://updates.jenkins-ci.org
COPY plugins.sh /usr/local/bin/plugins.sh
RUN chmod +x /usr/local/bin/plugins.sh

RUN apt-get update -y && \
  curl -sSL https://get.daocloud.io/docker | sh && \
  curl -L https://get.daocloud.io/docker/compose/releases/download/1.5.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose && \
  chmod +x /usr/local/bin/docker-compose

ENV java_version 1.8.0_11
ENV jdk_file jdk-8u11-linux-x64.tar.gz
ENV downloadlink http://download.oracle.com/otn-pub/java/jdk/8u11-b12/$jdk_file
ENV JAVA_HOME /opt/java-oracle/jdk$java_version
ENV PATH $JAVA_HOME/bin:$PATH
RUN wget --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" -O /tmp/$jdk_file $downloadlink && \
  mkdir /opt/java-oracle && tar -zxf /tmp/$jdk_file -C /opt/java-oracle/ && \
  update-alternatives --install /usr/bin/java java $JAVA_HOME/bin/java 20000 && \
  update-alternatives --install /usr/bin/javac javac $JAVA_HOME/bin/javac 20000 && \
  rm -rf /tmp/*

ENV MAVEN_HOME /opt/maven
RUN wget --no-verbose -O /tmp/apache-maven-3.2.2.tar.gz http://archive.apache.org/dist/maven/maven-3/3.2.2/binaries/apache-maven-3.2.2-bin.tar.gz && \
  echo "87e5cc81bc4ab9b83986b3e77e6b3095 /tmp/apache-maven-3.2.2.tar.gz" | md5sum -c && \
  tar xzf /tmp/apache-maven-3.2.2.tar.gz -C /opt/ && \
  ln -s /opt/apache-maven-3.2.2 /opt/maven && \
  ln -s /opt/maven/bin/mvn /usr/local/bin && \
  rm -f /tmp/apache-maven-3.2.2.tar.gz

RUN set -ex && \
  for key in \
    9554F04D7259F04124DE6B476D5A82AC7E37093B \
    94AE36675C464D64BAFA68DD7434390BDBE9B9C5 \
    0034A06D9D9B0064CE8ADF6BF1747F4AD2306D93 \
    FD3A5288F042B6850C66B31F09FE44734EB7990E \
    71DCFD284A79C3B38668286BC97EC7A07EDE3FC1 \
    DD8F2338BAE7501E3DD5AC78C273792F7D83545D \
  ; do \
    gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$key"; \
  done

ENV NPM_CONFIG_LOGLEVEL info
ENV NODE_VERSION 5.1.0
RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.gz" && \
  curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" && \
  gpg --verify SHASUMS256.txt.asc && \
  grep " node-v$NODE_VERSION-linux-x64.tar.gz\$" SHASUMS256.txt.asc | sha256sum -c - && \
  tar -xzf "node-v$NODE_VERSION-linux-x64.tar.gz" -C /usr/local --strip-components=1 && \
  rm "node-v$NODE_VERSION-linux-x64.tar.gz" SHASUMS256.txt.asc

ENV RUBY_MAJOR 2.2
ENV RUBY_VERSION 2.2.3
ENV RUBY_DOWNLOAD_SHA256 df795f2f99860745a416092a4004b016ccf77e8b82dec956b120f18bdc71edce
RUN echo 'install: --no-document\nupdate: --no-document' >> "$HOME/.gemrc" && \
  apt-get update -y && \
  apt-get install -y --force-yes zlib1g-dev libssl-dev bison libgdbm-dev ruby && \
  rm -rf /var/lib/apt/lists/* && \
  mkdir -p /usr/src/ruby && \
  curl -fSL -o ruby.tar.gz "http://cache.ruby-lang.org/pub/ruby/$RUBY_MAJOR/ruby-$RUBY_VERSION.tar.gz" && \
  echo "$RUBY_DOWNLOAD_SHA256 *ruby.tar.gz" | sha256sum -c - && \
  tar -xzf ruby.tar.gz -C /usr/src/ruby --strip-components=1 && \
  rm ruby.tar.gz && \
  cd /usr/src/ruby && \
  autoconf && \
  ./configure --disable-install-doc && \
  make -j"$(nproc)" && \
  make install && \
  apt-get purge -y --force-yes --auto-remove bison libgdbm-dev ruby && \
  rm -r /usr/src/ruby

ENV RUBYGEMS_VERSION 2.5.0
ENV BUNDLER_VERSION 1.10.6
ENV GEM_HOME /usr/local/bundle
ENV PATH $GEM_HOME/bin:$PATH
ENV BUNDLE_APP_CONFIG $GEM_HOME
RUN gem sources --add https://ruby.taobao.org/ --remove https://rubygems.org/ && \
  gem update --system $RUBYGEMS_VERSION && \
  gem install bundler --version "$BUNDLER_VERSION" && \
  bundle config --global path "$GEM_HOME" && \
  bundle config --global bin "$GEM_HOME/bin"

ADD http://mirrors.jenkins-ci.org/war/latest/jenkins.war /opt/jenkins.war
RUN chmod 644 /opt/jenkins.war

ENTRYPOINT ["java", "-jar", "-Duser.timezone=Asia/Shanghai", "/opt/jenkins.war"]
EXPOSE 8080

CMD [""]