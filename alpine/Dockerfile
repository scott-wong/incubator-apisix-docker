FROM openresty/openresty:alpine-fat

ARG APISIX_VERSION=1.1
LABEL apisix_version="${APISIX_VERSION}"

RUN echo "https://mirrors.aliyun.com/alpine/v3.9/main/" > /etc/apk/repositories \
    && echo "https://mirrors.aliyun.com/alpine/v3.9/community/" >> /etc/apk/repositories \
    && apk add --no-cache --virtual .builddeps \
    automake \
    autoconf \
    libtool \
    pkgconfig \
    cmake \
    git \
    && luarocks install https://raw.githubusercontent.com/apache/incubator-apisix/master/rockspec/apisix-${APISIX_VERSION}-0.rockspec --tree=/usr/local/apisix/deps \
    && cp /usr/local/apisix/deps/lib/luarocks/rocks-5.1/apisix/${APISIX_VERSION}-0/bin/apisix /usr/bin/ \
    && bin='#! /usr/local/openresty/luajit/bin/luajit\npackage.path = "/usr/local/apisix/lua/?.lua;" .. package.path' \
    && sed -i "1s@.*@$bin@" /usr/bin/apisix \
    && mv /usr/local/apisix/deps/share/lua/5.1/apisix/lua /usr/local/apisix \
    && apk del .builddeps \
    # 添加dashboard
    && cd /usr/local/apisix \
    && wget https://github.com/scott-wong/incubator-apisix-dashboard/releases/download/1.1/dist.tar.gz \
    && tar zxf dist.tar.gz \
    && rm -f dist.tar.gz \
    && mv dist dashboard

WORKDIR /usr/local/apisix

EXPOSE 9080 9443

CMD ["sh", "-c", "/usr/bin/apisix init && /usr/bin/apisix init_etcd && /usr/local/openresty/bin/openresty -p /usr/local/apisix -g 'daemon off;'"]
