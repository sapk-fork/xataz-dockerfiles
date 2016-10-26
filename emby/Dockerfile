FROM alpine:3.4
MAINTAINER xataz <https://github.com/xataz>

ARG MEDIAINFO_VER=0.7.88

ENV GID=991 \
    UID=991

LABEL description="Emby based on alpine" \
      tags="latest" \
      build_ver="2016100101"

RUN export BUILD_DEPS="build-base \
                        git \
                        wget \
			rpm@community \
                        ca-certificates" \
    && echo "@community http://dl-cdn.alpinelinux.org/alpine/edge/community" >> /etc/apk/repositories \
    && apk add -U imagemagick \
	            sqlite sqlite-dev \
	            ffmpeg \
	            s6 \
                su-exec \
                $BUILD_DEPS \
    && apk add -U --repository http://dl-3.alpinelinux.org/alpine/edge/testing/ mono \
    && wget http://mediaarea.net/download/binary/mediainfo/${MEDIAINFO_VER}/MediaInfo_CLI_${MEDIAINFO_VER}_GNU_FromSource.tar.gz -O /tmp/MediaInfo_CLI_${MEDIAINFO_VER}_GNU_FromSource.tar.gz \
    && wget http://mediaarea.net/download/binary/libmediainfo0/${MEDIAINFO_VER}/MediaInfo_DLL_${MEDIAINFO_VER}_GNU_FromSource.tar.gz -O /tmp/MediaInfo_DLL_${MEDIAINFO_VER}_GNU_FromSource.tar.gz \
    && tar xzf /tmp/MediaInfo_DLL_${MEDIAINFO_VER}_GNU_FromSource.tar.gz -C /tmp \
    && tar xzf /tmp/MediaInfo_CLI_${MEDIAINFO_VER}_GNU_FromSource.tar.gz -C /tmp \
    && cd  /tmp/MediaInfo_DLL_GNU_FromSource \
    && ./SO_Compile.sh \
    && cd /tmp/MediaInfo_DLL_GNU_FromSource/ZenLib/Project/GNU/Library \
    && make install \
    && cd /tmp/MediaInfo_DLL_GNU_FromSource/MediaInfoLib/Project/GNU/Library \
    && make install \
    && cd /tmp/MediaInfo_CLI_GNU_FromSource \
    && ./CLI_Compile.sh \
    && cd /tmp/MediaInfo_CLI_GNU_FromSource/MediaInfo/Project/GNU/CLI \
    && make install \
    && mkdir /embyServer /embyData \
    && curl -o /tmp/emby-server.rpm "http://ftp5.gwdg.de/pub/opensuse/repositories/home:/emby/openSUSE_Tumbleweed/x86_64/emby-server-3.0.8300-85.4.x86_64.rpm" \
    && rpm -i --notriggers --noscripts --nodeps --nosignature --force /tmp/emby-server.rpm \
    && cp -a /usr/lib/emby-server/bin/* /embyServer && ls /embyServer \
    && sed -i 's/ProgramData-Server/\/embyData/' /embyServer/MediaBrowser.Server.Mono.exe.config \
    && apk del $BUILD_DEPS \
    && rm -rf $(rpm -qlp /tmp/emby-server.rpm) /tmp/emby-server.rpm /var/cache/apk/* /tmp/*

EXPOSE 8096 8920 7359/udp
VOLUME /embyData
ADD rootfs /
RUN chmod +x /usr/local/bin/startup /etc/s6.d/*/*

ENTRYPOINT ["/usr/local/bin/startup"]
CMD ["s6-svscan", "/etc/s6.d"]
