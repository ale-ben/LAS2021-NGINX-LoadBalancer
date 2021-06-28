FROM nginx:1.20.0
RUN echo -e "\t\tCopying Config"
COPY Contents/nginx.conf /etc/nginx/nginx.conf
RUN echo -e "\t\tCopying WebServer"
COPY Contents/website /www/data
RUN echo -e "\t\Setting Permissions"
RUN chmod -R 555 /www/data/.
RUN chmod 555 /etc/nginx/nginx.conf