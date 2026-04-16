FROM php:8.3-fpm-bookworm

ARG UID=1000
ARG GID=1000

# ─── System dependencies ────────────────────────────────────────────────────
RUN apt-get update && apt-get install -y --no-install-recommends \
    # Build tools
    build-essential \
    autoconf \
    pkg-config \
    # Compression & archive
    unzip \
    zip \
    # Image/PDF support
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libpng-dev \
    libwebp-dev \
    libxpm-dev \
    # XML / intl / zip
    libxml2-dev \
    libicu-dev \
    libzip-dev \
    # Network / curl
    libcurl4-openssl-dev \
    # Node.js (for Vite builds)
    curl \
    gnupg \
    # Misc
    git \
    && rm -rf /var/lib/apt/lists/*

# ─── Node.js 20 (for Vite) ──────────────────────────────────────────────────
RUN apt-get update && apt-get install -y --no-install-recommends \
    nodejs npm && \
    rm -rf /var/lib/apt/lists/*

# ─── PHP extensions ─────────────────────────────────────────────────────────
# GD (for PDF rendering)
RUN docker-php-ext-configure gd \
        --with-freetype \
        --with-jpeg \
        --with-webp \
    && docker-php-ext-install -j$(nproc) gd

# Standard extensions
RUN docker-php-ext-install -j$(nproc) \
    bcmath \
    intl \
    opcache \
    pdo \
    pdo_mysql \
    zip \
    xml \
    pcntl \
    sockets

# Redis via PECL
RUN pecl install redis-6.0.2 && docker-php-ext-enable redis

# ─── Composer ────────────────────────────────────────────────────────────────
COPY --from=composer:2.7 /usr/bin/composer /usr/bin/composer

# ─── PHP configuration ──────────────────────────────────────────────────────
COPY php.ini      /usr/local/etc/php/conf.d/99-custom.ini
COPY opcache.ini  /usr/local/etc/php/conf.d/10-opcache.ini

# ─── Non-root user ──────────────────────────────────────────────────────────
RUN groupadd -g ${GID} appgroup && \
    useradd -u ${UID} -g appgroup -m -s /bin/bash appuser && \
    mkdir -p /var/www/html && \
    chown -R appuser:appgroup /var/www/html

USER appuser
WORKDIR /var/www/html

EXPOSE 9000
CMD ["php-fpm"]
