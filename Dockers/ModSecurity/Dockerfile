FROM debian:buster-slim as build
# Same as nginx:1.17.10

LABEL version="1.0.2"
LABEL maintainer="Manuel Nader <mnader@uoc.edu>"

RUN apt-get update \
 && apt-get install -y --no-install-recommends \
      automake \
      cmake \
      doxygen \
      g++ \
      git \
      libcurl4-gnutls-dev \
      libgeoip-dev \
      liblua5.3-dev \
      libpcre++-dev \
      libtool \
      libxml2-dev \
      make \
      ruby \
      wget \
      && apt-get clean \
      && rm -rf /var/lib/apt/lists/*

WORKDIR /sources

# Get ModSecurity files
RUN git clone https://github.com/SpiderLabs/ModSecurity --branch v3.0.4 --depth 1 \
 && cd ModSecurity \
 && ./build.sh \
 && git submodule init \
 && git submodule update

# Configure, make & install ModSecurity
RUN cd ModSecurity \
 && ./configure \
 && make \
 && make install

FROM nginx:1.17.10

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ENV ACCESSLOG=/var/log/nginx/access.log \
    BACKEND=http://localhost:80 \
    DNS_SERVER= \
    ERRORLOG=/var/log/nginx/error.log \
    LOGLEVEL=warn \
    METRICS_ALLOW_FROM='all' \
    METRICS_DENY_FROM='127.100.0.0/32' \
    #METRICS_ALLOW_FROM='127.0.0.0/24' \
    #METRICS_DENY_FROM='all' \
    METRICSLOG=/dev/null \
    MODSEC_AUDIT_LOG_TYPE=Serial \
    MODSEC_AUDIT_LOG=/dev/stdout \
    MODSEC_AUDIT_STORAGE=/var/log/modsecurity/audit/ \
    MODSEC_DATA_DIR=/tmp/modsecurity/data \
    MODSEC_DEBUG_LOG=/dev/null \
    MODSEC_DEBUG_LOGLEVEL=0 \
    MODSEC_PCRE_MATCH_LIMIT_RECURSION=100000 \
    MODSEC_PCRE_MATCH_LIMIT=100000 \
    MODSEC_REQ_BODY_ACCESS=on \
    MODSEC_REQ_BODY_LIMIT=13107200 \
    MODSEC_REQ_BODY_NOFILES_LIMIT=131072 \
    MODSEC_RESP_BODY_ACCESS=on \
    MODSEC_RESP_BODY_LIMIT=1048576 \
    MODSEC_RULE_ENGINE=on \
    MODSEC_TAG=modsecurity \
    MODSEC_TMP_DIR=/tmp/modsecurity/tmp \
    MODSEC_UPLOAD_DIR=/tmp/modsecurity/upload \
    PORT=80 \
    PROXY_TIMEOUT=60s \
    SERVER_NAME=localhost \
    TIMEOUT=60s \
    WORKER_CONNECTIONS=1024 \
    LD_LIBRARY_PATH=/lib:/usr/lib:/usr/local/lib

RUN apt-get update \
 && apt-get install -y --no-install-recommends \
      ca-certificates \
      gcc \
      git \
      libcurl4-gnutls-dev \
      liblua5.3-dev \
      libpcre++-dev \
      libxml2-dev \
      make \
      wget \
      moreutils \
      zlib1g-dev && \
      rm -rf /var/lib/apt/lists/* && \
      apt-get clean

RUN git clone https://github.com/SpiderLabs/ModSecurity-nginx --branch v1.0.1 --depth 1

COPY --from=build /usr/local/modsecurity/ /usr/local/modsecurity/
COPY --from=build /usr/local/lib/ /usr/local/lib/
# COPY --from=build /usr/share/TLS/server.key /etc/nginx/conf/server.key
# COPY --from=build /usr/share/TLS/server.crt /etc/nginx/conf/server.crt
# Move local files to repo
COPY src/etc/modsecurity.d/*.conf /etc/modsecurity.d/
COPY nginx/conf.d/*.conf /etc/nginx/conf.d/
COPY nginx/docker-entrypoint.sh /
COPY nginx/nginx.conf /etc/nginx/

# Copy Sites
COPY nginx/sites/ /usr/share/nginx/html

RUN version="$(/usr/sbin/nginx -v 2>&1 | cut -d '/' -f 2)" \
 && wget --quiet http://nginx.org/download/nginx-"$version".tar.gz \
 && tar -xvzf nginx-"$version".tar.gz \
 && cd /nginx-"$version" \
 && ./configure --with-compat --add-dynamic-module=../ModSecurity-nginx \
 && make modules \
 && cp objs/ngx_http_modsecurity_module.so /etc/nginx/modules/ \
 && wget --quiet https://raw.githubusercontent.com/SpiderLabs/ModSecurity/v3/master/modsecurity.conf-recommended \
    -O /etc/modsecurity.d/modsecurity.conf \
 && wget --quiet https://raw.githubusercontent.com/SpiderLabs/ModSecurity/v3/master/unicode.mapping \
    -O /etc/modsecurity.d/unicode.mapping

# OWASP ModSecurity Core Rule Set (CRS) v 3.2.0
RUN wget --quiet https://github.com/SpiderLabs/owasp-modsecurity-crs/archive/v3.2.0.tar.gz -O /etc/modsecurity.d/v3.2.0.tar.gz \
    && tar -xvzf /etc/modsecurity.d/v3.2.0.tar.gz -C /etc/modsecurity.d/ && rm /etc/modsecurity.d/v3.2.0.tar.gz

RUN chgrp -R 0 /var/cache/nginx/ /var/log/ /var/run/ /usr/share/nginx/ /etc/nginx/ /etc/modsecurity.d/ \
 && chmod -R g=u /var/cache/nginx/ /var/log/ /var/run/ /usr/share/nginx/ /etc/nginx/ /etc/modsecurity.d/

ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["nginx", "-g", "daemon off;"]
