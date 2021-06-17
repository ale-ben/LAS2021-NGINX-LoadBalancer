
FROM ubuntu:21.10

RUN echo -e "\t\Updating system and installing rsyslog" \
&& apt-get update && apt-get install rsyslog -y \

RUN echo -e "\t\tCopying Config"
COPY Contents/rsyslog.conf /etc/rsyslog.conf

ENTRYPOINT ["rsyslogd", "-n"]