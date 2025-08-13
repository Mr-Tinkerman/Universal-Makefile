# ======== For documentation, visit https://github.com/Mr-Tinkerman/Universal-Makefile/tree/master/docs/ ========


# ==========================================
# ======== BEGIN USER CONFIGURATION ========
# ==========================================

TARGET_FILE = target.exe

TOOLCHAIN_PREFIX = # Ex. xtensa-${PLATFORM}-elf-
PLATFORM = # Ex. esp32s3

BINARY_DIR     = bin/
BUILD_DIR      = build/
LIBRARY_DIR    = lib/

PUBLIC_API_DIR = include/

# SOURCE_DIRS = src/ src/core/
# SOURCE_FILES = # Ex: exclude/include_me.c

SOURCES = src/ src/core/

CFLAGS = -Wall -Werror -Wshadow -Wdouble-promotion -Wundef
LDFLAGS = -Wl,--gc-sections -fstack-protector -fsanitize=address

CFLAGS_Release = -O3
CFLAGS_Debug   = -O3 -g3

# Possible alternative to add targets
# $(eval $(call add_executable, NAME_EXE, SOURCES))
# $(eval $(call add_staticlibrary, NAMELIB, SOURCES))
# $(eval $(call add_sharedlibrary, NAMEDLL, SOURCES))
# $(eval $(call add_custom, NAMEDLL, TYPE, SOURCES))



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



# ======== Environment Detection ========

HOST_OS    := $(if $(filter Win%, ${OS}),WIN32, $(if $(filter Linux%, $(shell uname -s)),LINUX,$(if $(filter Darwin%, $(shell uname -s)),OSX),NULL_OS))

__PROC_GET := $(if $(filter WIN32, ${HOST_OS}),${PROCESSOR_ARCHITECTURE}${PROCESSOR_ARCHITEW6432},$(shell uname -p))
__IS_ARM   := $(if $(filter arm% ARM%, ${__PROC_GET}),ARM)

HOST_ARCH  := $(if $(filter %32 %64 %64T, ${__PROC_GET}),$(if ${__IS_ARM},ARM,x)$(if $(filter %64 %64T, ${__PROC_GET}),64,$(if ${__IS_ARM},32,86)),NULL_ARCH)

# Use not recommended outside of logging purposes
HOST_MACHINE := ${HOST_OS}, ${HOST_ARCH}



# ======== Variable Redeclaration ========

# Directory '/' Appending
TOOL_PATH      := $(if ${TOOL_PATH}, ${TOOL_PATH:/=}/,)
BINARY_DIR     := $(if ${BINARY_DIR}, ${BINARY_DIR:/=}/,)
BUILD_DIR      := $(if ${BUILD_DIR}, ${BUILD_DIR:/=}/,)
LIBRARY_DIR    := $(if ${LIBRARY_DIR}, ${LIBRARY_DIR:/=}/,)
PUBLIC_API_DIR := $(if ${PUBLIC_API_DIR}, ${PUBLIC_API_DIR:/=}/,)

# Linker Script Formatting
LINKER_SCRIPTS := $(if ${LINKER_SCRIPTS},$(foreach LS,${LINKER_SCRIPTS},${LS:%=-T%}))
LINKER_SCRIPTS_${CONFIG} := $(if ${LINKER_SCRIPTS_${CONFIG}},$(foreach LS,${LINKER_SCRIPTS_${CONFIG}},${LS:%=-T%}))

# Flag Consolidation
ARFLAGS  := ${ARFLAGS} ${ARFLAGS_${CONFIG}} rv
ASFLAGS  := ${ASFLAGS} ${ASFLAGS_${CONFIG}}
CFLAGS   := ${CFLAGS} ${CFLAGS_${CONFIG}}
CXXFLAGS := ${CXXFLAGS} ${CXXFLAGS_${CONFIG}} ${C_FLAGS}
LDFLAGS  := ${LDFLAGS} ${LDFLAGS_${CONFIG}} ${LINKER_SCRIPTS} ${LINKER_SCRIPTS_${CONFIG}}
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

# Format Embedded Commands
$(call _formatcmd,OBJCOPY)
$(call _formatcmd,MONITOR)
$(call _formatcmd,FLASH)

SOURCE_DIRS = $(filter %/,${SOURCES})
SOURCE_FILES = $(filter-out %/,${SOURCES})

OBJECTS = ${SOURCE_FILES} $(foreach SRC_DIR,${SOURCE_DIRS},$(wildcard ${SRC_DIR}*.c))
OBJECTS := $(sort ${OBJECTS:%.c=${BUILD_DIR}${CONFIG}/%.c.o})

REQUIRED_DIRS := ${SOURCE_DIRS} $(dir ${SOURCE_FILES})
REQUIRED_DIRS := ${REQUIRED_DIRS:%=${BUILD_DIR}${CONFIG}/%} ${BINARY_DIR}${CONFIG}/ ${LIBRARY_DIR} ${PUBLIC_API_DIR}



# ======== Build Rules ========

.SECONDARY:
.PHONY: all

all: ${BINARY_DIR}${CONFIG}/test

%/:
	${MKDIR} $@

${BUILD_DIR}${CONFIG}/%.S.o: %.S
	${AS} ${CFLAGS} -c $< -o $@

${BUILD_DIR}${CONFIG}/%.c.o: %.c
	${CC} ${CFLAGS} -c $< -o $@

${BUILD_DIR}${CONFIG}/%.cpp.o: %.cpp
	${CXX} ${CFLAGS} -c $< -o $@

${BINARY_DIR}${CONFIG}/test: ${REQUIRED_DIRS} ${OBJECTS}
	$(if $(filter 1,${OPT_USE_LDLINKING}),${LD},${CC}) ${LDFLAGS} ${OBJECTS} -o $@
