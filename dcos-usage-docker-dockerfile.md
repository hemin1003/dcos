## Dockerfile

### Dockerfile指令

**FROM：**

格式为 FROM&lt;image&gt; 或 FROM&lt;image&gt;:&lt;tag&gt;

第一条指令必须是FROM指令。并且，如果在同一个Dockerfile中创建多个镜像时，可以使用多个FROM指令（每个镜像一次）。

MAINTAINER：格式为MAINTAIER&lt;name&gt;，指定维护者信息。

**RUN：**

格式为RUN &lt;command&gt;或者RUN \[“executable”，“param1”，“param2”\]。

前者将在shell终端中运行的命令，即\/bin\/sh–c；后者则使用exec执行。指定使用其他终端可以通过第二种方式实现，例如RUN\[“\/bin\/bash”，“-c”，“echohello”\]。每条RUN指令将在当前镜像基础上执行指定命令，并提交为新的镜像。当命令较长时可以使用\来换行。

这实际上就是在容器构建时需要执行哪些指令，例如容器构建时需要下拉代码，但是默认启动的容器中是没有Git指令的，就需要下载，可以执行：RUN apt-get install -y git，然后RUN git clonexxxx

**CMD：******指定容器启动后执行的命令，一般都是早就写好的脚本，例如：CMD\[“\/run.sh”\]。注意：如果Dockerfile中指定了多条命令，只有最后一条会被执行。如果用户启动时候加了运行的命令，则会覆盖掉CMD指定的指令。

**EXPOSE：**

告诉Docker服务端容器需要暴露的端口号，供互联系统使用。在启动容器时需要通过-P（注意是大写），Docker主机会自动分配一个端口转发到指定的端口；使用-p，则可以具体指定哪个本地端口映射过来。

**ENV：**

1、创建的时候给容器中加上个需要的环境变量。2、指定一个值，为后续的RUN指令服务

**ADD：**

将复制指定的的文件复制到容器中。格式为 ADD &lt;src&gt; &lt;dest&gt; src必须为Dockerfile所在位置的相对路径，也可以是一个URL；还可以是一个tar文件（自动解压为目录）

**COPY：**

复制本地的文件或目录到容器中。目标路径不存在时，会自动创建。（和ADD类似，个人没发现啥区别）

**ENTRYPOINT：**

配置容器启动后执行的命令，并且不可被dockerrun 提供的参数覆盖。

每个Dockerfile中只能有一个ENTRYPOINT，当指定多个ENTRYPOINT时，只有最后一个生效。和CMD相似，却有不同。

**VOLUME：**

\[“\/data”\]创建一个挂在点，可以从本机或其他容器挂载的挂载点。意思就是从容器中暴露出一部分，和外界共享这块东西，一般放数据库的数据或者是代码。在容器启动运行的时候，如果需要将volume暴露的东西和本地的一个文件夹进行映射，想要通过本地文件直接访问容器中暴露的部分，可以在运行的时候进行映射：

docker run –v 本地路径：容器需要挂载的路径image 

但是有一个问题，在构建完毕第一次进行启动的时候，会以映射的本地环境为主，所以如果说本地环境为空，那么对应的容器中的文件将会变为空。

如果不指定本地的映射目录，那么docker会自动映射一个目录到本地（Mac和windows是被映射到docker machine中了），可以通过指令 docker inspect container\_name 来查看具体位置

**USER：**

指定运行容器时的用户名或者UID，后续的RUN也会使用指定的用户。当服务不需要管理员权限时，可以通过该命令指定运行用户。并且可以在之前创建所需要的用户。

要临时获取管理员权限的时候要使用gosu，不推荐使用sudo。如果不指定，容器默认是root运行。

**WORKDIR：**

定义工作目录，如果容器中没有此目录，会自动创建

**ONBUILD：**

配置当所创建的景象作为其他新创建景象的基础镜像时，所执行的操作指令。

例如，Dockerfile使用如下内容创建了镜像image-A

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

