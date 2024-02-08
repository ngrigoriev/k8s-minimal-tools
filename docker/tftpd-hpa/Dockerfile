FROM debian:bookworm-20240110 as build

ARG RSYSLOGD_TAG=v8.2312.0

RUN apt-get update && apt install -y git autoconf build-essential
RUN git clone git://git.kernel.org/pub/scm/network/tftp/tftp-hpa.git /usr/src/tftpd-hpa
WORKDIR /usr/src/tftpd-hpa

ADD configure.in.nogroups.diff /usr/src/tftpd-hpa/
# disable initgroups/setgroups - they will be problematic in distroless container without /etc/group
RUN patch < configure.in.nogroups.diff
RUN autoconf && autoupdate && autoreconf && ./configure --without-ipv6 --without-readline --without-tcpwrappers
RUN make SUB=tftpd && strip tftpd/tftpd

FROM gcr.io/distroless/base-nossl-debian12:nonroot
COPY --from=build /usr/src/tftpd-hpa/tftpd/tftpd /usr/sbin/
ADD tftpd.map /etc/tftpd-hpa/

USER nobody:nobody

# Not using --secure to avoid running privileged process.
# Using the mapping file to add /tftpboot/<client-ip> prefix instead

ENTRYPOINT ["/usr/sbin/tftpd", \
  "-4", \
  "--listen", \
  "--foreground", \
  "--address", "0.0.0.0:6969", \
  "--map-file", "/etc/tftpd-hpa/tftpd.map",\
  "--verbosity", "2", \
  "/tftproot"]
