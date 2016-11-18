## Dockerfile

### Dockerfile指令



### 示例（[docker-tomcat-base](https://github.com/christtrc/docker-tomcat-base)）

```
FROM isuper/java-oracle:jdk_8

ENV LANG zh_CN.UTF-8
ENV JAVA_HOME /usr/lib/jvm/java-8-oracle
ENV CATALINA_HOME /usr/local/tomcat
ENV PATH $CATALINA_HOME/bin:$PATH
RUN mkdir -p "$CATALINA_HOME"
WORKDIR $CATALINA_HOME

# let "Tomcat Native" live somewhere isolated
ENV TOMCAT_NATIVE_LIBDIR $CATALINA_HOME/native-jni-lib
ENV LD_LIBRARY_PATH ${LD_LIBRARY_PATH:+$LD_LIBRARY_PATH:}$TOMCAT_NATIVE_LIBDIR

# change timezone
RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \ 
    && echo "Asia/ShangHai" | tee /etc/timezone

# change APT sources.list to pull from Aliyun servers
# uncomment some entries and add some other entries
RUN sed -i "s/archive.ubuntu.com/mirrors.aliyun.com/" /etc/apt/sources.list
RUN apt-get update && apt-get install -y --no-install-recommends \ 
    libapr1 supervisor \ 
    && rm -rf /var/lib/apt/lists/* \ 
    && mkdir -p /var/log/supervisor

ENV TOMCAT_MAJOR 8
ENV TOMCAT_VERSION 8.5.8
ENV TOMCAT_TGZ_URL http://mirrors.aliyun.com/apache/tomcat/tomcat-$TOMCAT_MAJOR/v$TOMCAT_VERSION/bin/apache-tomcat-$TOMCAT_VERSION.tar.gz
# not all the mirrors actually carry the .asc files :'(
ENV TOMCAT_ASC_URL http://mirrors.aliyun.com/apache/tomcat/tomcat-$TOMCAT_MAJOR/v$TOMCAT_VERSION/bin/apache-tomcat-$TOMCAT_VERSION.tar.gz.asc

RUN set -x \ 
    \ 
    && curl -o tomcat.tar.gz "$TOMCAT_TGZ_URL" \ 
    && curl -o tomcat.tar.gz.asc "$TOMCAT_ASC_URL" \ 
    # && gpg --batch --verify tomcat.tar.gz.asc tomcat.tar.gz \ 
    && tar -xvf tomcat.tar.gz --strip-components=1 \ 
    && rm bin/*.bat \ 
    && rm -rf webapps/* \ 
    && rm tomcat.tar.gz* \ 
    \ 
    && nativeBuildDir="$(mktemp -d)" \ 
    && tar -xvf bin/tomcat-native.tar.gz -C "$nativeBuildDir" --strip-components=1 \ 
    && nativeBuildDeps=" \ 
        gcc \ 
        libapr1-dev \ 
        libssl-dev \ 
        make \ 
    " \ 
    && apt-get update && apt-get install -y --no-install-recommends $nativeBuildDeps && rm -rf /var/lib/apt/lists/* \ 
    && ( \ 
        export CATALINA_HOME="$PWD" \ 
        && cd "$nativeBuildDir/native" \ 
        && ./configure \ 
            --libdir="$TOMCAT_NATIVE_LIBDIR" \ 
            --prefix="$CATALINA_HOME" \ 
            --with-apr="$(which apr-1-config)" \ 
            --with-java-home="$JAVA_HOME" \ 
            --with-ssl=yes \ 
        && make -j$(nproc) \ 
        && make install \ 
    ) \ 
    && apt-get purge -y --auto-remove $nativeBuildDeps \ 
    && rm -rf "$nativeBuildDir" \ 
    && rm bin/tomcat-native.tar.gz

# verify Tomcat Native is working properly
RUN set -e \ 
    && nativeLines="$(catalina.sh configtest 2>&1)" \ 
    && nativeLines="$(echo "$nativeLines" | grep 'Apache Tomcat Native')" \ 
    && nativeLines="$(echo "$nativeLines" | sort -u)" \ 
    && if ! echo "$nativeLines" | grep 'INFO: Loaded APR based Apache Tomcat Native library' >&2; then \ 
        echo >&2 "$nativeLines"; \ 
        exit 1; \ 
    fi

RUN curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-5.0.1-amd64.deb \ 
    && dpkg -i filebeat-5.0.1-amd64.deb \ 
    && rm -rf filebeat-5.0.1-amd64.deb

COPY context.xml /usr/local/tomcat/conf

# COPY filebeat.yml /etc/filebeat/filebeat.yml
# COPY supervisord.conf /etc/supervisord.conf

EXPOSE 8080

CMD ["catalina.sh", "run"]

# CMD ["/usr/bin/supervisord"]

```

