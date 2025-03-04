FROM centos:7

MAINTAINER actanble <actanble@gmail.com>

# TODO Set the time and lang
ENV LANG en_US.UTF-8
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
RUN curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
RUN sed -i -e '/mirrors.cloud.aliyuncs.com/d' -e '/mirrors.aliyuncs.com/d' /etc/yum.repos.d/CentOS-Base.repo
RUN curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
RUN yum makecache

# All Libs Used Backup In https://github.com/offgithub On 2019-5-28
ENV TENGINE_VERSION=2.3.2
ENV DEPLOY_PATH=/nginx_waf

RUN mkdir -p ${DEPLOY_PATH} /spool/log/nginx/ /var/cache/nginx/
ADD . ${DEPLOY_PATH}

WORKDIR ${DEPLOY_PATH}

# TODO Install yum-libs Deppends
RUN yum -y install wget curl curl-devel git
RUN yum install autoconf automake bzip2 \
         flex httpd-devel libaio-devel yajl yajl-devel \
         libass-devel libjpeg-turbo-devel libpng12-devel \
         libtheora-devel libtool libva-devel libvdpau-devel \
         libvorbis-devel libxml2-devel libxslt-devel \
         perl texi2html unzip zip openssl lua lua-devel lmdb lmdb-devel \
         openssl-devel geoip geoip-devel ssdeep ssdeep-devel gcc-c++ \
         gd-devel GeoIP GeoIP-devel GeoIP-data doxygen libxml2 libxml2-dev zlib-devel \
         perl perl-devel perl-ExtUtils-Embed redhat-rpm-config -y

# Download Tengine
RUN wget https://github.com/alibaba/tengine/archive/${TENGINE_VERSION}.zip  && unzip ${TENGINE_VERSION}.zip -d ${DEPLOY_PATH}

# TODO Build Modsecurity
# https://github.com/SpiderLabs/ModSecurity/wiki/Compilation-recipes-for-v3.x#centos-7-minimal-dynamic

RUN git clone --depth 1 -b v3/master --single-branch https://github.com/SpiderLabs/ModSecurity \
    && cd ModSecurity \
    && git submodule init \
    && git submodule update \
    && ./build.sh \
    && ./configure \
    && make \
    && make install


RUN git clone --depth 1 https://github.com/SpiderLabs/ModSecurity-nginx.git

RUN cd tengine-${TENGINE_VERSION} && ./configure --with-compat --add-dynamic-module=../ModSecurity-nginx && \
make modules && cp objs/ngx_http_modsecurity_module.so /lib64

# TODO Main Build tengine+modsecurity
RUN cd tengine-${TENGINE_VERSION} && ./configure   ./configure --prefix=/etc/nginx \
	 --sbin-path=/usr/sbin/nginx \
	 --modules-path=/usr/lib64/nginx/modules \
	 --conf-path=/etc/nginx/nginx.conf \
	 --error-log-path=/spool/log/nginx/error.log \
	 --http-log-path=/spool/log/nginx/access.log \
	 --pid-path=/var/run/nginx.pid \
	 --lock-path=/var/run/nginx.lock \
	 --http-client-body-temp-path=/var/cache/nginx/client_temp \
	 --http-proxy-temp-path=/var/cache/nginx/proxy_temp \
	 --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
	 --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
	 --http-scgi-temp-path=/var/cache/nginx/scgi_temp \
	 --user=nginx --group=nginx \
	 --with-compat --with-file-aio \
	 --with-threads \
	 --with-http_addition_module \
	 --with-http_auth_request_module \
	 --with-http_dav_module \
	 --with-http_flv_module \
	 --with-http_gunzip_module \
	 --with-http_gzip_static_module \
	 --with-http_mp4_module \
	 --with-http_random_index_module \
	 --with-http_realip_module \
	 --with-http_secure_link_module \
	 --with-http_slice_module \
	 --with-http_ssl_module \
	 --with-http_stub_status_module \
	 --with-http_sub_module \
	 --with-http_v2_module \
	 --with-mail \
	 --with-mail_ssl_module \
	 --with-stream \
	 --with-stream_realip_module \
	 --with-stream_ssl_module \
	 --with-stream_ssl_preread_module \
	 --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -fPIC' \
	 --with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie'


RUN wget https://github.com/SpiderLabs/owasp-modsecurity-crs/archive/v3.1.0.zip && unzip v3.1.0.zip -d /opt/ && \
mv /opt/owasp-modsecurity-crs-3.1.0 /opt/owasp-modsecurity-crs

RUN echo -e "\033[46;30m *****Link some Libs****** \033[0m"
# TODO Add Default Conf Log Dir and Temp dir

RUN cp ${DEPLOY_PATH}/ModSecurity/unicode.mapping /etc/nginx ;
# RUN ln -sv `find / -type f -name "libprofiler.so*" | grep /usr/local/lib` /lib64/libprofiler.so.0
# RUN ln -sv `find / -type f -name "libluajit-5.1.so*" | grep /usr/local/lib` /lib64/libluajit-5.1.so.2
# RUN ln -sv `find / -type f -name "libmodsecurity.so*" | grep /usr/local/` /lib64/libmodsecurity.so.3

RUN useradd -s /sbin/nologin -M nginx

# TODO Add Supervisord Replaced tini
RUN yum -y install supervisor
COPY supervisor_ngx.conf /etc/supervisord.conf

# Todo Save the env and not DELTE
# RUN rm -rf ${DEPLOY_PATH}
WORKDIR /root/
USER root

EXPOSE 80
# HEALTHCHECK --interval=5m --timeout=3s CMD nginx || curl -f http://localhost/ || exit 1
CMD ["/usr/bin/supervisord", '-c', '/etc/supervisord.conf']

