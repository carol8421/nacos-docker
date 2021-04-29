FROM centos:7.5.1804
MAINTAINER pader "carol8421@126.com"

# set environment
ENV MODE="cluster" \
    PREFER_HOST_MODE="ip"\
    BASE_DIR="/home/nacos" \
    CLASSPATH=".:/home/nacos/conf:$CLASSPATH" \
    CLUSTER_CONF="/home/nacos/conf/cluster.conf" \
    FUNCTION_MODE="all" \
    JAVA_HOME="/usr/lib/jvm/java-1.8.0-openjdk" \
    NACOS_USER="nacos" \
    JAVA="/usr/lib/jvm/java-1.8.0-openjdk/bin/java" \
    JVM_XMS="1g" \
    JVM_XMX="1g" \
    JVM_XMN="512m" \
    JVM_MS="128m" \
    JVM_MMS="320m" \
    NACOS_DEBUG="n" \
    TOMCAT_ACCESSLOG_ENABLED="false" \
    TIME_ZONE="Asia/Shanghai"

ARG NACOS_VERSION=2.0.0
ARG HOT_FIX_FLAG=-bugfix

WORKDIR $BASE_DIR

RUN set -x \
    && yum update -y \
    && yum install -y java-1.8.0-openjdk java-1.8.0-openjdk-devel wget iputils nc  vim libcurl curl sudo dnf
RUN wget  https://github.com/alibaba/nacos/releases/download/${NACOS_VERSION}${HOT_FIX_FLAG}/nacos-server-${NACOS_VERSION}.tar.gz -P /home
RUN tar -xzvf /home/nacos-server-${NACOS_VERSION}.tar.gz -C /home \
    && rm -rf /home/nacos-server-${NACOS_VERSION}.tar.gz /home/nacos/bin/* /home/nacos/conf/*.properties /home/nacos/conf/*.example /home/nacos/conf/nacos-mysql.sql
RUN yum autoremove -y wget \
    && ln -snf /usr/share/zoneinfo/$TIME_ZONE /etc/localtime && echo $TIME_ZONE > /etc/timezone \
    && yum clean all

# RUN curl -s https://install.zerotier.com | sudo sh
FROM centos:7.5.1804 as builder
MAINTAINER Adam Ierymekno <adam.ierymenko@zerotier.com>, Grant Limberg <grant.limberg@zerotier.com>

ARG git_branch=master

RUN yum update -y
RUN yum update -y python*
RUN yum install -y dnf-data dnf-plugins-core libdnf-devel libdnf python2-dnf-plugin-migrate dnf-automatic
RUN yum -y install epel-release && yum -y update && yum clean all
RUN yum groupinstall -y "Development Tools"
RUN yum install -y bash postgresql10 postgresql10-devel libpqxx-devel clang jemalloc jemalloc-devel
RUN yum install -y curl sudo dnf
RUN yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm && dnf -qy module disable postgresql


# RUN git clone http://git.int.zerotier.com/zerotier/ZeroTierOne.git
# RUN if [ "$git_branch" != "master" ]; then cd ZeroTierOne && git checkout -b $git_branch origin/$git_branch; fi
ADD ./ZeroTierOne /ZeroTierOne
# RUN [ "echo $PWD" ]
RUN cd ZeroTierOne && make clean && make central-controller
# RUN cd ZeroTierOne && make central-controller

FROM centos:7.5.1804
RUN yum update -y python*
RUN yum install -y dnf-data dnf-plugins-core libdnf-devel libdnf python2-dnf-plugin-migrate dnf-automatic
RUN yum install -y jemalloc jemalloc-devel postgresql10
RUN yum install -y curl sudo dnf
RUN yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm && dnf -qy module disable postgresql && yum -y install epel-release && yum -y update && yum clean all


COPY --from=builder /ZeroTierOne/zerotier-one /usr/local/bin/zerotier-one
RUN chmod a+x /usr/local/bin/zerotier-one

ADD ZeroTierOne/ext/central-controller-docker/main.sh /
RUN chmod a+x /main.sh

EXPOSE 9993/udp

CMD [ "/main.sh" ]

ADD bin/docker-startup.sh bin/docker-startup.sh
ADD conf/application.properties conf/application.properties
ADD init.d/custom.properties init.d/custom.properties


# set startup log dir
RUN mkdir -p logs \
	&& cd logs \
	&& touch start.out \
	&& ln -sf /dev/stdout start.out \
	&& ln -sf /dev/stderr start.out
RUN chmod +x bin/docker-startup.sh

EXPOSE 8848
ENTRYPOINT ["bin/docker-startup.sh"]
