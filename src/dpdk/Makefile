# makefile to manage dpdk package
#

# Copyright 2015-2017 OPNFV
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

#
# Contributors:
#   Aihua Li, Huawei Technologies.
#   Martin Klozik, Intel Corporation.
#   Christian Trautman, Red Hat Inc.
#   Maryam Tahhan, Intel Corporation.

include ../mk/master.mk
include ../package-list.mk
include /etc/os-release

.PHONY: install force_make

WORK_DIR = dpdk
TAG_DONE_FLAG = $(WORK_DIR)/.$(DPDK_TAG).tag.done

# VHOST configuration options are stored in different files based on DPDK version
# v1.2.3r0-v1.6.0r2 - configuration inside config/defconfig_x86_64-default-linuxapp-gcc
# v1.7.0-rc1-v2.2.0 - configuration inside config/common_linuxapp
# v16 and newer - configuration split between config/common_linuxapp and config/common_base
DPDK_TAG_MAJOR = $(shell echo $(DPDK_TAG) | cut -d. -f1)
DPDK_TAG_MINOR = $(shell echo $(DPDK_TAG) | cut -d. -f2)
ifeq ($(DPDK_TAG_MAJOR),v1)
ifeq ($(DPDK_TAG_MINOR), $(filter $(DPDK_TAG_MINOR), 7 8))
    DPDK_TARGET = x86_64-native-linuxapp-gcc
    CONFIG_FILE_LINUXAPP = $(WORK_DIR)/config/common_linuxapp
else
    DPDK_TARGET = x86_64-default-linuxapp-gcc
    CONFIG_FILE_LINUXAPP = $(WORK_DIR)/config/defconfig_x86_64-default-linuxapp-gcc
endif
else
ifeq ($(DPDK_TAG_MAJOR),v2)
    DPDK_TARGET = x86_64-native-linuxapp-gcc
    CONFIG_FILE_LINUXAPP = $(WORK_DIR)/config/common_linuxapp
else
    DPDK_TARGET = x86_64-native-linuxapp-gcc
    CONFIG_FILE_BASE = $(WORK_DIR)/config/common_base
    CONFIG_FILE_LINUXAPP = $(WORK_DIR)/config/common_linuxapp
endif
endif

all: force_make install
	@echo "Finished making $(WORK_DIR) "

INSTALL_TARGET = force_make

# modify CONFIG_FILE to enable SHARED_LIB build and restore original CONFIG_FILE after the build
# DPDK v16 comments:
## CONFIG_RTE_BUILD_COMBINE_LIBS has been obsoleted
## CONFIG_RTE_LIBRTE_KNI is listed in both config_base and config_linuxapp,
## values from config_linuxapp will be used, but options are modified at both places to avoid confusion.
force_make: $(TAG_DONE_FLAG)
	$(AT)cd $(WORK_DIR) && git pull $(DPDK_URL) $(DPDK_TAG)
ifdef CONFIG_FILE_BASE
	$(AT)sed -i -e 's/CONFIG_RTE_BUILD_SHARED_LIB=./CONFIG_RTE_BUILD_SHARED_LIB=y/g' $(CONFIG_FILE_BASE)
	$(AT)sed -i -e 's/CONFIG_RTE_EAL_PMD_PATH=.*/CONFIG_RTE_EAL_PMD_PATH=\"\/usr\/lib\/dpdk-pmd\/\"/g' $(CONFIG_FILE_BASE)
	$(AT)sed -i -e 's/CONFIG_RTE_LIBRTE_KNI=./CONFIG_RTE_LIBRTE_KNI=n/g' $(CONFIG_FILE_BASE)
else
	$(AT)sed -i -e 's/CONFIG_RTE_BUILD_SHARED_LIB=./CONFIG_RTE_BUILD_SHARED_LIB=y/g' $(CONFIG_FILE_LINUXAPP)
	$(AT)sed -i -e 's/CONFIG_RTE_EAL_PMD_PATH=.*/CONFIG_RTE_EAL_PMD_PATH=\"\/usr\/lib\/dpdk-pmd\/\"/g' $(CONFIG_FILE_LINUXAPP)
	$(AT)sed -i -e 's/CONFIG_RTE_BUILD_COMBINE_LIBS=./CONFIG_RTE_BUILD_COMBINE_LIBS=y/g' $(CONFIG_FILE_LINUXAPP)
endif
# do not build kernel module under docker
ifdef DOCKER
	$(AT)sed -i -e 's/CONFIG_RTE_EAL_IGB_UIO=y/CONFIG_RTE_EAL_IGB_UIO=n/' $(CONFIG_FILE_LINUXAPP)
	$(AT)sed -i -e 's/CONFIG_RTE_KNI_KMOD=y/CONFIG_RTE_KNI_KMOD=n/' $(CONFIG_FILE_LINUXAPP)
endif
# CentOS 7.3 specific config changes to compile
ifeq ($(ID),"centos")
ifeq ($(VERSION_ID),"7")
	$(AT)sed -i.bak s@'SRCS-y += ethtool/igb/igb_main.c'@'#SRCS-y += ethtool/igb/igb_main.c'@g $(WORK_DIR)/lib/librte_eal/linuxapp/kni/Makefile
endif
endif
# RHEL 7.3 specific config changes to compile
ifeq ($(ID),"rhel")
ifeq ($(VERSION_ID),"7.3")
	$(AT)sed -i.bak s@'SRCS-y += ethtool/igb/igb_main.c'@'#SRCS-y += ethtool/igb/igb_main.c'@g $(WORK_DIR)/lib/librte_eal/linuxapp/kni/Makefile
endif
endif
	$(AT)sed -i -e 's/CONFIG_RTE_LIBRTE_KNI=./CONFIG_RTE_LIBRTE_KNI=n/g' $(CONFIG_FILE_LINUXAPP)
	$(AT)cd $(WORK_DIR); make config T=$(DPDK_TARGET) -j; make -j DESTDIR=$(WORK_DIR)
	$(AT)cd `dirname $(CONFIG_FILE_LINUXAPP)` && git checkout `basename $(CONFIG_FILE_LINUXAPP)` && cd -
	@echo "Make done"

install: $(INSTALL_TARGET)
	$(AT)cd $(WORK_DIR); sudo make -j install prefix=/usr; cd -
	$(AT)sudo mkdir -p /usr/lib/dpdk-pmd
	$(AT)find /usr/lib -type f -name 'librte_pmd*' | while read path; do sudo ln -fs $$path /usr/lib/dpdk-pmd/$$(echo $$path | grep -o 'librte_.*so'); done
# under docker there won't be kernel module available
ifndef DOCKER
	$(AT)sudo cp -a $(WORK_DIR)/build/kmod $(INSTALL_DIR)/lib/modules/$(KERNEL_VERSION)
endif
	@echo "install done"

# hard way to clean and clobber
clean:
	$(AT)cd $(WORK_DIR) && git clean -xfd *.o
clobber:
	$(AT)rm -rf $(WORK_DIR)

# distclean is for developer who would like to keep the
# clone git repo, saving time to fetch again from url
distclean:
	$(AT)cd $(WORK_DIR) && git clean -xfd && git checkout -f

test:
	@echo "Make test in $(WORK_DIR) (stub) "

sanity:
	@echo "Make sanity in $(WORK_DIR) (stub) "

$(WORK_DIR):
	$(AT)git clone $(DPDK_URL)

$(TAG_DONE_FLAG): $(WORK_DIR)
	$(AT)cd $(WORK_DIR); git checkout $(DPDK_TAG)
	$(AT)touch $@
