#
# In most cases, you only need to modify this first section.
#
LIB = libtmctl.so

OBJS = tmctl_api.o

OBJS += tmctl_api_trace.o


all install: conditional_build

clean: generic_clean
	rm -f $(INSTALL_DIR)/lib/private/$(LIB)



#
# Set our CommEngine directory (by splitting the pwd into two words
# at /userspace and taking the first word only).
# Then include the common defines under CommEngine.
# You do not need to modify this part.
#
CURR_DIR := $(shell pwd)
BUILD_DIR:=$(HND_SRC)

include $(BUILD_DIR)/make.common


ifneq ($(strip $(BUILD_FAPCTL)),)
OBJS += tmctl_fap.o
OBJS += tmctl_api_fap.o
endif

ifneq ($(strip $(BUILD_RDPACTL)),)
OBJS += tmctl_rdpa.o tmctl_ethsw.o
ifneq ($(strip $(BUILD_DSL)),)
OBJS += tmctl_xtm.o
endif
OBJS += tmctl_api_runner.o
ifeq ($(strip $(DESKTOP_LINUX)),)
OBJS += tmctl_bdmf_rdpa.o
endif
endif

ifeq ($(strip $(BRCM_CHIP)),63381)
ifneq ($(strip $(BUILD_BCMTM)),)
OBJS += tmctl_bcmtm.o
OBJS += tmctl_api_softtm.o
endif
endif

ifneq ($(strip $(BUILD_ARCHERCTL)),)
OBJS += tmctl_archer.o tmctl_ethsw.o
OBJS += tmctl_api_archer.o
endif

ifneq ($(strip $(BUILD_TMCTL)),)
conditional_build: generic_private_lib_install
else
conditional_build:
	@echo "skipping $(LIB) (not configured)"
endif

#
# Private apps and libs are allowed to include header files from the
# private and public directories
#
# WARNING: Do not modify this section unless you understand the
# license implications of what you are doing.
#
ALLOWED_INCLUDE_PATHS := -I.\
                         -I$(BUILD_DIR)/userspace/public/include  \
                         -I$(BUILD_DIR)/userspace/public/include/$(OALDIR) \
                         -I$(BCM_FSBUILD_DIR)/public/include \
                         -I$(BUILD_DIR)/userspace/private/include  \
                         -I$(BUILD_DIR)/userspace/private/include/$(OALDIR) \
                         -I$(BUILD_DIR)/shared/opensource/include/bcm963xx \
                         -I$(BUILD_DIR)/shared/private/include/$(OALDIR) \
                         -I$(INC_BRCMDRIVER_PRIV_PATH)/$(BRCM_BOARD) \
                         -I$(INC_BRCMDRIVER_PUB_PATH)/$(BRCM_BOARD) 

ifneq ($(strip $(BUILD_RDPACTL)),)
ALLOWED_INCLUDE_PATHS += -I$(BUILD_DIR)/rdp/drivers/rdpa_gpl/include/ \
                         -I$(BUILD_DIR)/rdp/drivers/bdmf/framework/ \
                         -I$(PROJECT_DIR)/target/bdmf/system \
                         -I$(PROJECT_DIR)/target/bdmf/system/sim \
                         -I$(PROJECT_DIR)/target/rdpa_gpl \
                         -I$(PROJECT_DIR)/target/rdpa_user \
                         -I$(PROJECT_DIR)/target/rdpa_user/include \
                         -I$(PROJECT_DIR)/target/rdpa_user/include/autogen/autogen 
endif


# For customers who manually dlopen libtmctl.so, we need to explicitly
# list the library dependencies so they are listed as DT_NEEDED in this lib
ifneq ($(strip $(BUILD_FAPCTL)),)
LOCAL_LDFLAGS :=  -L$(INSTALL_DIR)/lib -lfapctl
CFLAGS += -DSUPPORT_FAPCTL
endif
ifneq ($(strip $(BUILD_RDPACTL)),)
LOCAL_LDFLAGS :=  -L$(INSTALL_DIR)/lib -lrdpactl -lethswctl -lbdmf 
ifneq ($(strip $(BUILD_DSL)),)
LOCAL_LDFLAGS += -latmctl
CFLAGS += -DBUILD_DSL
endif
endif
ifeq ($(strip $(BRCM_CHIP)),63381)
ifneq ($(strip $(BUILD_BCMTM)),)
LOCAL_LDFLAGS :=  -L$(INSTALL_DIR)/lib -lbcmtm
CFLAGS += -DSUPPORT_BCMTM
endif
endif
ifneq ($(strip $(BUILD_EPONCTL)),)
CFLAGS += -DSUPPORT_FLUSH
LOCAL_LDFLAGS += -L$(INSTALL_DIR)/lib -leponctl
ALLOWED_INCLUDE_PATHS += -I$(BUILD_DIR)/userspace/private/libs/eponctl
endif

#
# Implicit rule will make the .c into a .o
# Implicit rule is $(CC) -c $(CPPFLAGS) $(CFLAGS)
# See Section 10.2 of Gnu Make manual
# 
$(LIB):
	echo "Copying $@"
	-cp -f $(TOP_PLATFORM)/$(shell pwd | sed 's/\(.*router-sysdep\/\)//')/prebuilt/$@ ./



#
# Include the rule for making dependency files.
# The '-' in front of the second include suppresses
# error messages when make cannot find the .d files.
# It will just regenerate them.
# See Section 4.14 of Gnu Make.
#

include $(BUILD_DIR)/make.deprules

-include $(OBJS:.o=.d)
