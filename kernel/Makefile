ifndef TARGET_COMPILE
    $(error TARGET_COMPILE not set)
endif

# ========== 添加输出目录支持 ==========
ifeq ($(O),)
    OUT_DIR := .
else
    OUT_DIR := $(O)
endif

TARGET := $(OUT_DIR)/kpimg
TARGET_ELF := $(OUT_DIR)/kpimg.elf

CC = $(TARGET_COMPILE)gcc
LD = $(TARGET_COMPILE)ld
AS = $(TARGET_COMPILE)as
OBJCOPY = $(TARGET_COMPILE)objcopy

CFLAGS += -Wall -fno-builtin -std=gnu11 -nostdinc -mgeneral-regs-only
CFLAGS += -g

ifdef DEBUG
	CFLAGS += -DDEBUG -DMAP_DEBUG -g
endif

ifdef ANDROID
	CFLAGS += -DANDROID
endif

INCLUDE := -I. -Iinclude -Ipatch/include -Ilinux -Ilinux/include -Ilinux/arch/arm64/include -Ilinux/tools/arch/arm64/include

BASE_SRCS += base/setup.c 
BASE_SRCS += base/setup1.S
BASE_SRCS += base/cache.S
BASE_SRCS += base/tlsf.c
BASE_SRCS += base/start.c 
BASE_SRCS += base/map.c 
BASE_SRCS += base/map1.S 
BASE_SRCS += base/hook.c 
BASE_SRCS += base/fphook.c 
BASE_SRCS += base/hotpatch.c 
BASE_SRCS += base/hmem.c 
BASE_SRCS += base/predata.c 
BASE_SRCS += base/symbol.c 
BASE_SRCS += base/baselib.c 
BASE_SRCS += base/sha256.c 

BASE_SRCS += $(wildcard patch/*.c)
BASE_SRCS += $(filter-out patch/common/supercall.c patch/common/supercmd.c, $(wildcard patch/common/*.c))
BASE_SRCS += $(wildcard patch/module/*.c)
BASE_SRCS += $(wildcard patch/ksyms/*.c)

ifdef ANDROID
	BASE_SRCS += $(wildcard patch/android/*.c)
endif

SRCS += $(BASE_SRCS)
SRCS += $(LINUX_SRCS)

# 修改：将 .o 文件输出到输出目录
OBJS := $(SRCS:.c=.o)
OBJS := $(OBJS:.S=.o)
OBJS := $(addprefix $(OUT_DIR)/, $(OBJS))

# 确保输出目录存在
$(shell mkdir -p $(OUT_DIR) $(OUT_DIR)/base $(OUT_DIR)/patch $(OUT_DIR)/patch/common $(OUT_DIR)/patch/module $(OUT_DIR)/patch/ksyms)

all: hdr ${TARGET}

${TARGET}: ${TARGET_ELF}
	${OBJCOPY} -O binary -S $^ $@

${TARGET_ELF}: ${OBJS}
	${LD} -nostdlib -static -no-pie -Tkpimg.lds -o $@ $^

# 修改：编译 .c 文件到输出目录
$(OUT_DIR)/%.o: %.c
	mkdir -p $(dir $@)
	${CC} $(CFLAGS) $(INCLUDE) -c -O2 -o $@ $<

# 特殊处理 sha256
$(OUT_DIR)/base/sha256.o: base/sha256.c
	mkdir -p $(dir $@)
	${CC} $(CFLAGS) $(INCLUDE) -c -O0 -o $@ $<

# 修改：编译 .S 汇编文件到输出目录
$(OUT_DIR)/%.o: %.S
	mkdir -p $(dir $@)
	${CC} $(CFLAGS) $(INCLUDE) -c -o $@ $<

.PHONY: hdr
hdr:
	mkdir -p $(OUT_DIR)/user
	cp -Rf patch/include/uapi $(OUT_DIR)/user/
	cp -f ../version $(OUT_DIR)/user/
	cp -f include/preset.h $(OUT_DIR)/tools/ 2>/dev/null || true

.PHONY: clean
clean:
	rm -rf $(OUT_DIR)/*.elf
	rm -rf $(OUT_DIR)/kpimg
	rm -rf $(OUT_DIR)/base $(OUT_DIR)/patch