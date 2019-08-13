FROM php:7.3-fpm

MAINTAINER Andrew Beak <andy.beak@gmail.com>

# Use official recommendation for production configuration
RUN mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"

# Copy our PHP source code in
COPY src/ /src

# Set file permissions
RUN chown -R www-data:www-data /src

# Expose port 9000 to containers on our network
EXPOSE 9000

# Start up the php-fpm process
CMD ["php-fpm"]