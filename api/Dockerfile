# Use the PHP base image
FROM docker.io/library/php:8.1-fpm

# Install dependencies
RUN apt-get update \
    && apt-get install -y \
       curl \
       git \
    && rm -rf /var/lib/apt/lists/*

# Install Composer globally
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Set working directory
WORKDIR /var/www/html

# Copy application files
COPY . /var/www/html

# Install PHP dependencies using Composer
RUN composer install --no-interaction --optimize-autoloader

# Expose port (if necessary)
# EXPOSE 8000

# Start PHP-FPM server
CMD ["php-fpm"]

