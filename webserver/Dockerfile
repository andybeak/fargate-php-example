FROM nginx:stable

# Copy the server declaration configuration
COPY config/default.conf /etc/nginx/conf.d/default.conf

# Copy example static content
COPY static_content/ /var/www/html
RUN chown -R nginx:nginx /var/www/html

# Expose ports
EXPOSE 80
EXPOSE 443

# Define working directory.
WORKDIR /etc/nginx

# Define default command.
CMD ["nginx", "-g", "daemon off;"]