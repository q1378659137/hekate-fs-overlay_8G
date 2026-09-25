ifeq ($(strip $(DEVKITARM)),)
$(error "Please set DEVKITARM in your environment. export DEVKITARM=<path to>devkitARM")
endif

include $(DEVKITARM)/base_rules

################################################################################

LDR_LOAD_ADDR := 0x40007000
IPL_MAGIC := 0x43544349 #"ICTC"
include ../Versions.inc

################################################################################

TARGET := loader
BUILDDIR := ../build/$(TARGET)
OUTPUTDIR := ../output
BDKDIR := bdk
BDKINC := -I../$(BDKDIR)
VPATH += $(dir $(wildcard ../$(BDKDIR)/*/)) $(dir $(wildcard ../$(BDKDIR)/*/*/))

# Main and graphics.
OBJS = $(addprefix $(BUILDDIR)/, \
	start.o loader.o lz.o \
)

################################################################################

CUSTOMDEFINES := -DBL_MAGIC=$(IPL_MAGIC)
CUSTOMDEFINES += -DBL_VER_MJ=$(BLVERSION_MAJOR) -DBL_VER_MN=$(BLVERSION_MINOR) -DBL_VER_HF=$(BLVERSION_HOTFX) -DBL_VER_RL=$(BLVERSION_REL)

# Force the 8GB DRAM ID. This is the payload that gets injected/chainloaded,
# so its reserved config is the one that ends up in the running image.
# Must be kept in sync with the top-level Makefile. See there for details.
ifeq ($(RAM8GB),1)
CUSTOMDEFINES += -DFORCE_8GB_DRAM_ID
endif

#TODO: Considering reinstating some of these when pointer warnings have been fixed.
WARNINGS := -Wall -Wsign-compare -Wno-array-bounds -Wno-stringop-overflow

ARCH := -march=armv4t -mtune=arm7tdmi -mthumb-interwork
CFLAGS = $(ARCH) -O2 -g -nostdlib -ffunction-sections -fdata-sections -fomit-frame-pointer -std=gnu11 $(WARNINGS) $(CUSTOMDEFINES)
LDFLAGS = $(ARCH) -nostartfiles -lgcc -Wl,--nmagic,--gc-sections -Xlinker --defsym=LDR_LOAD_ADDR=$(LDR_LOAD_ADDR)

################################################################################

.PHONY: all clean

all: $(TARGET).bin $(TOOLSLZ) $(TOOLSB2C)

clean:
	@rm -f payload_00.h
	@rm -f payload_01.h

$(TARGET).bin: $(BUILDDIR)/$(TARGET).elf
	@$(OBJCOPY) -S -O binary $< $(OUTPUTDIR)/$(PAYLOAD_NAME).bin

$(BUILDDIR)/$(TARGET).elf: $(OBJS)
	@$(CC) $(LDFLAGS) -T link.ld $^ -o $@

$(BUILDDIR)/%.o: %.c
	@$(CC) $(CFLAGS) $(BDKINC) -c $< -o $@

$(BUILDDIR)/%.o: %.S
	@$(CC) $(CFLAGS) -c $< -o $@

$(OBJS): $(BUILDDIR)

$(BUILDDIR):
	@mkdir -p "$(BUILDDIR)"
	@mkdir -p "$(OUTPUTDIR)"
