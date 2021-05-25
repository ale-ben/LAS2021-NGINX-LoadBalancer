FROM ubuntu:latest
RUN echo -e "\t\tUpdating APT"
RUN apt update
RUN echo -e "\t\tInstalling packages"
RUN apt-get install -y curl \
					wget \
					iputils-ping \
					vim \
					tmux

ENTRYPOINT ["/bin/bash"]