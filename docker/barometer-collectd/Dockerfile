FROM centos:7
RUN yum update -y && \
        yum install -y which sudo git && \
        yum clean all && \
        git config --global http.sslVerify false

ENV DOCKER y
ENV WITH_DPDK y
ENV COLLECTD_FLAVOR stable
ENV repos_dir /src
ENV openstack_plugins /src/barometer/src/collectd-openstack-plugins

WORKDIR ${repos_dir}
RUN git clone https://gerrit.opnfv.org/gerrit/barometer
WORKDIR ${repos_dir}/barometer/systems
RUN sh ./build_base_machine.sh && \
        useradd -ms /bin/bash collectd_exec && \
        echo "collectd_exec ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

WORKDIR ${openstack_plugins}
RUN make && \
        pip install --upgrade pip && \
        pip install -r ${openstack_plugins}/collectd-openstack-plugins/requirements.txt

COPY run_collectd.sh /run_collectd.sh
RUN chmod +x /run_collectd.sh

ENTRYPOINT ["/run_collectd.sh"]
