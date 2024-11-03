FROM php:8.2-apache-bullseye

# Suppress user dialogs (and selects the default answer) during apt-get install
ARG DEBIAN_FRONTEND=noninteractive

# Update packages
RUN apt-get update \
    && apt-get -y upgrade \
    && rm -rf /var/lib/apt/lists/*

# Configure default server name
RUN echo "ServerName localhost" | tee /etc/apache2/conf-available/fqdn.conf
RUN a2enconf fqdn

# Enable apache modules
RUN a2enmod rewrite headers
