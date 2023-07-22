# Use a PHP 8 base image
FROM nvidia/cuda:12.2.0-devel-ubuntu20.04 as builder
ENV DEBIAN_FRONTEND=noninteractive
# Install CUDA and cuBLAS dependencies
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
    curl \
    libxpm-dev \
    libfreetype6-dev \
    gnupg

# Install cuBLAS
RUN apt-get update && apt-get install -y libcublas-11-4

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
    libreadline-dev

# Download and extract PHP source code
WORKDIR /usr/src/php
RUN apt-get install -y wget pkg-config && \
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
    --enable-gd \
    --with-xpm \
    --with-freetype \
    --with-readline \
    --with-zip \
    && make -j$(nproc) \
    && make install

# Base image for the final PHP runtime
FROM nvidia/cuda:12.2.0-runtime-ubuntu20.04 as runtime
ENV DEBIAN_FRONTEND=noninteractive
# Copy the compiled PHP installation from the builder stage
COPY --from=builder /usr/local/php /usr/local/php

# Set the environment variables
ENV PATH="/usr/local/php/bin:${PATH}"
ENV PHP_INI_SCAN_DIR="/usr/local/php/etc/conf.d"

# Set the environment variables for CUDA
ENV PATH="/usr/local/php/bin:${PATH}"
ENV PHP_INI_SCAN_DIR="/usr/local/php/etc/conf.d"
ENV LD_LIBRARY_PATH="/usr/local/cuda/lib64:${LD_LIBRARY_PATH}"

# Install necessary CUDA and cuBLAS dependencies
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
    libcublas-12-1 \
    nvidia-cuda-toolkit

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
    liblapacke-dev \
    libfreetype6-dev \
    git

# Install necessary dependencies
RUN apt-get update && apt-get install -y \
    git \
    libopenblas-dev


# Set the working directory
WORKDIR /src

RUN git clone https://github.com/NumPower/numpower

WORKDIR /src/numpower

# Compile NumPower with CUDA enabled
RUN phpize && ./configure --with-cuda && make install-cuda

RUN mkdir -p /usr/local/php/etc && mkdir -p /usr/local/php/etc/conf.d
RUN echo 'extension=ndarray.so' > /usr/local/php/etc/conf.d/ndarray.ini

# Start the PHP CLI
CMD ["/bin/bash"]