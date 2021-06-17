
FROM ubuntu

RUN echo -e "\t\Updating system and installing rsyslog"
RUN apt update && apt install rsyslog -y

RUN echo -e "\t\tCopying Config"
COPY Contents/rsyslog.conf /etc/rsyslog.conf

ENTRYPOINT ["rsyslogd", "-n"]