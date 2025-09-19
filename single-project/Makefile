# ======== For documentation, visit https://github.com/Mr-Tinkerman/Universal-Makefile/tree/master/docs/ ========


# ==========================================
# ======== BEGIN USER CONFIGURATION ========
# ==========================================

TARGET_EXECUTABLES := TARGEX

TARGEX_REALNAME := targetexample
TARGEX_SOURCES := src/ src/app/

DEFAULT_TARGETS := TARGEX

SOURCES := src/core/

CFLAGS = -Wall -Werror -Wshadow -Wdouble-promotion -Wundef
LDFLAGS = -Wl,--gc-sections -fstack-protector -fsanitize=address

CFLAGS_Release = -O3
CFLAGS_Debug   = -O3 -g3

# TOOLCHAIN_PREFIX = xtensa-${PLATFORM}-elf-
# PLATFORM = esp32s3



# ==========================================
# ======== END USER CONFIGURATION ==========
# ==========================================


# ======== Functions ========

define _formatcmd
$(eval $1_INSIDE  := $(wordlist 2, $(words $(wordlist 2, $(words ${$1}), ${$1})), ${$1}))
$(eval $1_FW      := $(firstword ${$1}))
$(eval $1_LW      := $(if $(filter ${$1_FW}, $(lastword ${$1})),,$(lastword ${$1})))
$(eval $1_QFILTER := $(filter "%",${$1_FW}${$1_LW}))

$(eval $1_FW      := $(if $(filter 1, $(words ${$1})),${$1_FW:%"=%},${$1_FW}))

$(eval $1 := $(if ${$1_QFILTER},${$1_FW:"%=%} ${$1_INSIDE} ${$1_LW:%"=%}, \
        ${TOOL_PATH}${TOOLCHAIN_PREFIX}${$1_FW}${TOOLCHAIN_SUFFIX} ${$1_INSIDE} ${$1_LW}))
$(eval $1 := $(strip ${$1}))
endef

define _gentargetprereqs
$(eval $1_REALNAME ?= $1)
$(eval $1_OUTFILE := ${$1_REALNAME}$2)
$(eval TARGETFILES += ${$1_OUTFILE})

$(eval $1_SOURCES := ${SOURCES} ${$1_SOURCES} ${$1_SOURCES_${ARCH}} ${$1_SOURCES_${CONFIG}} ${$1_SOURCES_${CONFIG}_${ARCH}})

$(eval $1_SOURCE_DIRS = $(filter %/,${$1_SOURCES}))
$(eval $1_SOURCE_FILES = $(filter-out %/,${$1_SOURCES}))

$(eval $1_OBJECTS := $(patsubst %,${BUILD_DIR}${CONFIG}/%.o,${$1_SOURCE_FILES} $(filter ${SRC_EXTS},$(foreach SRC_DIR,${$1_SOURCE_DIRS},$(wildcard ${SRC_DIR}*)))))
$(eval ${$1_OUTFILE}_OBJECTS := ${$1_OBJECTS})

$(eval $1_REQDIRS := $(addprefix ${BUILD_DIR}${CONFIG}/,${$1_SOURCE_DIRS} $(dir ${$1_SOURCE_FILES})) ${REQDIRS})
endef



# ======== Environment Detection ========

HOST_OS    := $(if $(filter Win%, ${OS}),WIN32, $(if $(filter Linux%, $(shell uname -s)),LINUX,$(if $(filter Darwin%, $(shell uname -s)),OSX),NULL_OS))

__PROC_GET := $(if $(filter WIN32, ${HOST_OS}),${PROCESSOR_ARCHITECTURE}${PROCESSOR_ARCHITEW6432},$(shell uname -p))
__IS_ARM   := $(if $(filter arm% ARM%, ${__PROC_GET}),ARM)

HOST_ARCH  := $(if $(filter %32 %64 %64T, ${__PROC_GET}),$(if ${__IS_ARM},ARM,x)$(if $(filter %64 %64T, ${__PROC_GET}),64,$(if ${__IS_ARM},32,86)),NULL_ARCH)

# Use not recommended outside of logging purposes
HOST_MACHINE := ${HOST_OS}, ${HOST_ARCH}



# ======== Caching ========

-include cache.mk

# Verify $(CONFIG) is set or cached
CONFIG := $(if ${CONFIG},${CONFIG},${LAST_CONFIG})
$(info ${CONFIG})
$(info ${LAST_CONFIG})
$(info $(if ${CONFIG},Exists,Null))



# ======== Variable Transformation ========

# Directory '/' Appending
TOOL_PATH      := $(if ${TOOL_PATH}, ${TOOL_PATH:/=}/,)
BINARY_DIR     := $(if ${BINARY_DIR}, ${BINARY_DIR:/=}/,bin/)
BUILD_DIR      := $(if ${BUILD_DIR}, ${BUILD_DIR:/=}/,build/)
LIBRARY_DIR    := $(if ${LIBRARY_DIR}, ${LIBRARY_DIR:/=}/,lib/)
PUBLIC_API_DIR := $(if ${PUBLIC_API_DIR}, ${PUBLIC_API_DIR:/=}/,include/)

# Linker Script Formatting
LINKER_SCRIPTS_${CONFIG} := $(addprefix -T ,${LINKER_SCRIPTS_${CONFIG}:-T%=%})
LINKER_SCRIPTS := $(addprefix -T ,${LINKER_SCRIPTS:-T%=%}) ${LINKER_SCRIPTS_${CONFIG}}

# Flag Consolidation
ARFLAGS  := ${ARFLAGS} ${ARFLAGS_${CONFIG}} rv
ASFLAGS  := ${ASFLAGS} ${ASFLAGS_${CONFIG}}
CFLAGS   := ${CFLAGS} ${CFLAGS_${CONFIG}}
CXXFLAGS := ${CXXFLAGS} ${CXXFLAGS_${CONFIG}}
LDFLAGS  := ${LDFLAGS} ${LDFLAGS_${CONFIG}} ${LINKER_SCRIPTS}
PPFLAGS  := ${PPFLAGS} ${PPFLAGS_${CONFIG}}

LDLIBS   := ${LDLIBS} ${LDLIBS_${CONFIG}}

# Set Command Defaults
MKDIR ?= "${TOOL_PATH}mkdir -p"
COPY  ?= "${TOOL_PATH}cp"
DEL   ?= "${TOOL_PATH}${RM}"

OBJCOPY ?= "${TOOL_PATH}objcopy"

# Format System Commands
$(call _formatcmd,MKDIR)
$(call _formatcmd,COPY)
$(call _formatcmd,DEL)

RM := ${DEL}

# Format Compilation Commands
$(call _formatcmd,AR)
$(call _formatcmd,AS)
$(call _formatcmd,CC)
$(call _formatcmd,CXX)
$(call _formatcmd,LD)
$(call _formatcmd,PP)

CPP := $(if ${PP},${PP},${CC} -E)
LD  := $(if $(filter 1,${OPT_USE_LDLINKING}),${LD},${CC})

# Format Embedded Commands
$(call _formatcmd,OBJCOPY)
$(call _formatcmd,MONITOR)
$(call _formatcmd,FLASH)

# Consolidate Source Extensions
SRC_EXTS := .S .c .cpp ${SRC_EXTS}
SRC_EXTS := $(foreach EXT,${SRC_EXTS},%${EXT})



# ======== Target / Prereq Generation ========

ALL_TARGETS     := ${TARGET_EXECUTABLES} ${TARGET_BINARIES} ${TARGET_STATICLIBS} ${TARGET_SHAREDLIBS}
DEFAULT_TARGETS := ${TARGET_EXECUTABLES} ${TARGET_BINARIES}

# Objects and Directory List Generation from Sources
SOURCES := ${SOURCES} ${SOURCES_${ARCH}} ${SOURCES_${CONFIG}} ${SOURCES_${CONFIG}_${ARCH}}
REQDIRS := ${BINARY_DIR}${CONFIG}/ ${LIBRARY_DIR} ${PUBLIC_API_DIR}

$(foreach T, ${TARGET_EXECUTABLES}, $(call _gentargetprereqs,${T},.exe))
$(foreach T, ${TARGET_STATICLIBS},  $(call _gentargetprereqs,${T},.a))
$(foreach T, ${TARGET_SHAREDLIBS},  $(call _gentargetprereqs,${T},.so))
$(foreach T, ${TARGET_BINARIES},    $(call _gentargetprereqs,${T},.bin))



# ======== Build Rules ========

.SECONDEXPANSION:
.SECONDARY:
.PHONY: all default ${ALL_TARGETS}

default: ${DEFAULT_TARGETS}

all: ${ALL_TARGETS}

%/:
	${MKDIR} $@

${BUILD_DIR}${CONFIG}/%.S.o: %.S
	${AS} ${CFLAGS} $< -o $@

${BUILD_DIR}${CONFIG}/%.c.o: %.c
	${CC} ${CFLAGS} -c $< -o $@

${BUILD_DIR}${CONFIG}/%.cpp.o: %.cpp
	${CXX} ${CFLAGS} -c $< -o $@

${ALL_TARGETS}: $${$$@_REQDIRS} $${$$@_TARGETDEPS} $${$$@_OUTFILE}

${TARGETFILES}: $${$$@_OBJECTS}
	${LD} ${LDFLAGS} $< -o $@
