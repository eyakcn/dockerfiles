FROM fluent/fluentd:latest

USER ubuntu
WORKDIR /home/ubuntu
ENV PATH /home/ubuntu/ruby/bin:$PATH

RUN gem sources --add https://ruby.taobao.org/ --remove https://rubygems.org/ && \
    gem update --system $RUBYGEMS_VERSION && \
    gem install fluent-plugin-secure-forward && \
    gem install fluent-plugin-s3

COPY fluent.conf /fluentd/etc/
RUN chmod -R +rw /fluentd/log

CMD fluentd -c /fluentd/etc/$FLUENTD_CONF -p /fluentd/plugins $FLUENTD_OPT