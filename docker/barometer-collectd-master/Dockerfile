FROM centos:7
RUN yum update -y && \
        yum install -y which sudo git && \
        yum clean all && \
        git config --global http.sslVerify false

ENV DOCKER y
ENV COLLECTD_FLAVOR master
ENV WITH_DPDK y
ENV repos_dir /src

WORKDIR ${repos_dir}
RUN git clone https://github.com/cloud-guru/barometer-opnfv
WORKDIR ${repos_dir}/barometer-opnfv/systems
RUN sh ./build_base_machine.sh && \
        useradd -ms /bin/bash collectd_exec && \
        echo "collectd_exec ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

COPY run_collectd.sh /run_collectd.sh
RUN chmod +x /run_collectd.sh

ENTRYPOINT ["/run_collectd.sh"]
