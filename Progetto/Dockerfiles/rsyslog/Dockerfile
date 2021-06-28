FROM ubuntu:21.10

RUN echo -e "\t\Updating system and installing rsyslog" \
&& apt-get update \
&& apt-get install --no-install-recommends -y rsyslog \
&& apt-get clean \
&& rm -rf /var/lib/apt/lists/*

RUN echo -e "\t\tCopying Config"
COPY Contents/rsyslog.conf /etc/rsyslog.conf

ENTRYPOINT ["rsyslogd", "-n"]