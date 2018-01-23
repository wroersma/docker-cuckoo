FROM blacktop/volatility:2.6

ENV CUCKOO_VERSION 2.0.5.1
ENV CUCKOO_CWD /cuckoo
ENV SSDEEP 2.14.1


COPY requirements.txt /tmp/requirements.txt

RUN apk update
RUN apk add --no-cache ca-certificates \
                       libconfig \
                       libpq \
                       zlib \
                       py-lxml \
                       py-pillow \
                       py-libvirt \
                       py-chardet \
                       py-crypto \
                       openssl \
                       openssh-client \
                       curl

RUN apk add --no-cache -t .build-deps \
                           gcc \
                           g++ \
                           python-dev \
                           libpq \
                           py-pip \
                           linux-headers \
                           libxslt-dev \
                           libxml2-dev \
                           pcre-dev \
                           python-dev \
                           libffi-dev \
                           build-base \
                           libstdc++ \
                           zlib-dev \
                           libc-dev \
                           jpeg-dev \
                           file-dev \
                           automake \
                           autoconf \
                           libtool \
                           py-pip \
                           postgresql-dev \
                           libconfig-dev \
                           libmagic \
                           python \
                           swig \
                           git

RUN set -x \
  && echo "===> Install ssdeep..." \
  && curl -Ls https://github.com/ssdeep-project/ssdeep/releases/download/release-$SSDEEP/ssdeep-$SSDEEP.tar.gz > \
    /tmp/ssdeep-$SSDEEP.tar.gz \
  && cd /tmp \
  && tar xzf ssdeep-$SSDEEP.tar.gz \
  && cd ssdeep-$SSDEEP \
  && ./configure \
  && make \
  && make install \
  && echo "===> Install pydeep..." \
  && cd /tmp \
  && git clone https://github.com/kbandla/pydeep.git \
  && cd pydeep \
  && python setup.py build \
  && python setup.py install \
  && echo "===> Install Cuckoo Sandbox..." \
  && mkdir /cuckoo \
  && export PIP_NO_CACHE_DIR=off \
  && export PIP_DISABLE_PIP_VERSION_CHECK=on \
  && pip install --upgrade pip wheel setuptools


COPY ./cuckoo /tmp/cuckoo

RUN cd /tmp/cuckoo \
  && pip install psycopg2 \
  && LDFLAGS=-L/lib python stuff/monitor.py \
  && LDFLAGS=-L/lib python setup.py sdist install

RUN cd /tmp/cuckoo \
  && LDFLAGS=-L/lib pip install . \
  && LDFLAGS=-L/lib cuckoo \
  && LDFLAGS=-L/lib cuckoo community \
  && echo "===> Install additional dependencies..." \
  && pip install -r /tmp/requirements.txt \
  && apk add --update tcpdump \
  && apk add suricata \
  && pip install m2crypto==0.24.0 \
  && cd /tmp \
  && git clone https://github.com/seanthegeek/etupdate.git \
  && cp etupdate/etupdate /usr/sbin \
  && /usr/sbin/etupdate -V \
  && echo "===> Clean up unnecessary files..." \
  && rm -rf /tmp/* \
  && apk del --purge .build-deps



COPY conf /cuckoo/conf
COPY update_conf.py /update_conf.py
COPY docker-entrypoint.sh /entrypoint.sh

WORKDIR /cuckoo

VOLUME ["/cuckoo/conf"]

EXPOSE 1337 31337

ENTRYPOINT ["/entrypoint.sh"]
CMD ["--help"]
