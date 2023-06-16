FROM ubuntu:22.04 as builder
ENV DEBIAN_FRONTEND=noninteractive

# Install required build dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    autoconf \
    libssl-dev \
    libxml2-dev \
    libsqlite3-dev \
    zlib1g-dev \
    libonig-dev \
    libbz2-dev \
    libcurl4-openssl-dev \
    libjpeg-dev \
    libpng-dev \
    libzip-dev \
    libwebp-dev \
    libxpm-dev \
    libfreetype6-dev \
    liblapack-dev \
    liblapacke-dev \
    pkg-config \
    git \
    libopenblas-dev

# Download and extract PHP source code
WORKDIR /usr/src/php
RUN apt-get install -y wget && \
    wget -O php.tar.gz https://www.php.net/distributions/php-8.2.0.tar.gz && \
    tar -xf php.tar.gz --strip-components=1 && \
    rm php.tar.gz

# Configure and compile PHP with desired options
RUN ./configure \
    --prefix=/usr/local/php \
    --with-config-file-path=/usr/local/php/etc \
    --enable-mbstring \
    --with-curl \
    --with-openssl \
    --with-libxml \
    --with-pdo-mysql \
    --with-pdo-sqlite \
    --with-sqlite3 \
    --with-zlib \
    --with-bz2 \
    --with-jpeg \
    --with-webp \
    --with-xpm \
    --with-freetype \
    --with-zip \
    && make -j$(nproc) \
    && make install

# Base image for the final PHP runtime
FROM ubuntu:22.04 as runtime
ENV DEBIAN_FRONTEND=noninteractive
# Copy the compiled PHP installation from the builder stage
COPY --from=builder /usr/local/php /usr/local/php

# Set the environment variables
ENV PATH="/usr/local/php/bin:${PATH}"
ENV PHP_INI_SCAN_DIR="/usr/local/php/etc/conf.d"

RUN apt-get update && apt-get install -y \
    build-essential \
    autoconf \
    libssl-dev \
    libxml2-dev \
    libsqlite3-dev \
    zlib1g-dev \
    libonig-dev \
    libbz2-dev \
    libcurl4-openssl-dev \
    libjpeg-dev \
    libpng-dev \
    pkg-config \
    libzip-dev \
    libwebp-dev \
    libxpm-dev \
    libfreetype6-dev \
    liblapack-dev \
    liblapacke-dev \
    git \
    libopenblas-dev \
    file

# Set the working directory
WORKDIR /src

RUN git clone https://github.com/NumPower/numpower.git /src

RUN phpize && \
    ./configure && \
    make -j$(nproc) && \
    make install

RUN mkdir /usr/local/php/conf.d/
RUN echo "extension=ndarray.so" > /usr/local/php/conf.d/ndarray.ini

# Start the PHP CLI
CMD ["/bin/bash"]