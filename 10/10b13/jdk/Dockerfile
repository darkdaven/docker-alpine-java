# AlpineLinux with a glibc-2.28-r0 and Oracle Java 8
FROM alpine:3.7

LABEL MAINTAINER="Jose Ariel Herrera <darkdaven87@gmail.com>"
# thanks to Vladimir Krivosheev <develar@gmail.com> aka @develar for smaller image
# and Victor Palma <palma.victor@gmail.com> aka @devx for pointing it out
# and Anastas Dancha <anapsix@random.io> this is a fork of his version of java 8

# Java Version and other ENV
ENV JAVA_VERSION_MAJOR=10 \
    JAVA_VERSION_MINOR=0.2 \
    JAVA_VERSION_MINOR_CHECKSUM=0-2 \
    JAVA_VERSION_BUILD=13 \
    JAVA_PACKAGE=jdk \
    JAVA_JCE=standard \
    JAVA_HOME=/opt/jdk \
    PATH=${PATH}:/opt/jdk/bin \
    GLIBC_REPO=https://github.com/sgerrand/alpine-pkg-glibc \
    GLIBC_VERSION=2.28-r0 \
    LANG=C.UTF-8

# do all in one step
RUN set -ex
RUN command [[ ${JAVA_VERSION_MAJOR} != 7 ]] || ( echo >&2 'Oracle no longer publishes JAVA7 packages' && exit 1 )
RUN apk -U upgrade
RUN apk add libstdc++ curl ca-certificates bash
RUN for pkg in glibc-${GLIBC_VERSION} glibc-bin-${GLIBC_VERSION} glibc-i18n-${GLIBC_VERSION}; do curl -sSL ${GLIBC_REPO}/releases/download/${GLIBC_VERSION}/${pkg}.apk -o /tmp/${pkg}.apk; done
RUN apk add --allow-untrusted /tmp/*.apk
RUN rm -v /tmp/*.apk
RUN ( /usr/glibc-compat/bin/localedef --force --inputfile POSIX --charmap UTF-8 C.UTF-8 || true )
RUN echo "export LANG=C.UTF-8" > /etc/profile.d/locale.sh
RUN /usr/glibc-compat/sbin/ldconfig /lib /usr/glibc-compat/lib
RUN mkdir /opt
RUN curl -jksSLH "Cookie: oraclelicense=accept-securebackup-cookie" -o /tmp/java.tar.gz \
    http://download.oracle.com/otn-pub/java/jdk/${JAVA_VERSION_MAJOR}.${JAVA_VERSION_MINOR}+${JAVA_VERSION_BUILD}/19aef61b38124481863b1413dce1855f/${JAVA_PACKAGE}-${JAVA_VERSION_MAJOR}.${JAVA_VERSION_MINOR}_linux-x64_bin.tar.gz
RUN JAVA_PACKAGE_SHA256=$(curl -sSL https://www.oracle.com/webfolder/s/digest/${JAVA_VERSION_MAJOR}-${JAVA_VERSION_MINOR_CHECKSUM}checksum.html | grep "${JAVA_PACKAGE}-${JAVA_VERSION_MAJOR}.${JAVA_VERSION_MINOR}-linux-x64_bin.tar.gz" | grep -Eo '(sha256: )[^<]+' | cut -d: -f2 | xargs)
RUN echo "${JAVA_PACKAGE_SHA256} /tmp/java.tar.gz" > /tmp/java.tar.gz.sha256
RUN sha256sum -c /tmp/java.tar.gz.sha256
RUN gunzip /tmp/java.tar.gz
RUN tar -C /opt -xf /tmp/java.tar
RUN ln -s /opt/jdk-${JAVA_VERSION_MAJOR}.${JAVA_VERSION_MINOR} /opt/jdk
RUN apk del curl glibc-i18n
RUN rm -rf /tmp/* /var/cache/apk/*
RUN echo 'hosts: files mdns4_minimal [NOTFOUND=return] dns mdns4' >> /etc/nsswitch.conf
# EOF