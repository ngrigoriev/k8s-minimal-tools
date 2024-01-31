FROM debian:bookworm-20240110 as build

ARG RSYSLOGD_TAG=v8.2312.0

RUN apt-get update && apt install -y build-essential pkg-config libestr-dev libfastjson-dev zlib1g-dev flex bison git autoconf automake libtool python3-docutils
RUN git clone https://github.com/rsyslog/rsyslog.git /usr/src/rsyslog
WORKDIR /usr/src/rsyslog
RUN git checkout $RSYSLOGD_TAG
ENV CONF_FILE_PATH=/etc/rsyslogd/rsyslogd.conf
ENV PID_FILE_PATH=/tmp/rsyslogd/rsyslogd.pid
RUN autoreconf -fiv
RUN ./configure --disable-largefile \
--disable-libgcrypt \
--disable-klog \
--disable-kmsg \
--disable-debug \
--disable-debug-symbols \
--disable-libgcrypt \
--disable-uuid \
--disable-openssl \
--disable-libcrypt \
--disable-gnutls \
--disable-gnutls-tests \
--disable-libzstd \
--disable-generate-man-pages \
--disable-fmhttp \
--runstatedir=/tmp/rsyslogd \
--localstatedir=/tmp/rsyslogd \
--prefix=/usr \
--sysconfdir=/etc/rsyslogd

RUN make && strip tools/rsyslogd && make install prefix=/usr/src/rsyslog/dist
# copy native libraries needed by the binary
RUN mkdir -p dist/lib && cp $(ldd dist/sbin/rsyslogd | grep '=>' | grep -v 'lib[cm]' | awk '{print $3}') dist/lib

FROM gcr.io/distroless/base-nossl-debian12:nonroot
COPY --from=build /usr/src/rsyslog/dist/sbin/rsyslogd /usr/sbin/
#RUN mkdir -p /usr/lib/rsyslog /etc/rsyslogd/rsyslog.d
COPY --from=build /usr/src/rsyslog/dist/lib/rsyslog/* /usr/lib/rsyslog/
# copy dependencies
COPY --from=build /usr/src/rsyslog/dist/lib/* /lib/

ADD rsyslogd.conf /etc/rsyslogd/
ADD 99-promtail-relay.conf /etc/rsyslogd/rsyslog.d/

# some defaults
ENV LISTEN_PORT=50514
ENV FORWARD_HOST=localhost
ENV FORWARD_PORT=514
ENV FORWARD_PROTO=udp

CMD ["/usr/sbin/rsyslogd", "-n"]
