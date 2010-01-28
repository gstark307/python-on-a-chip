# PyMite Configuration
PLATFORM := $(notdir $(CURDIR))
PM_LIB_ROOT = pmvm_$(PLATFORM)
PM_LIB_FN = lib$(PM_LIB_ROOT).a
PM_LIB_PATH = ../../vm/$(PM_LIB_FN)
PM_USR_SOURCES = main.py
PMIMGCREATOR := ../../tools/pmImgCreator.py
IPM = true
DEBUG = false
# If I wasn't lazy, I'd add these to my path. But I'm lazy.
CC = /cygdrive/c/Program\ Files/Microchip/MPLAB\ C30/bin/pic30-gcc.exe
AR = /cygdrive/c/Program\ Files/Microchip/MPLAB\ C30/bin/pic30-ar.exe
B2H = /cygdrive/c/Program\ Files/Microchip/MPLAB\ C30/bin/pic30-bin2hex.exe
CPU = 24HJ64GP502
# Per docs, the PIC$(CPU) define should already exist. Not sure why this is necessary.
# If running in simulation, include the -DSIM below; otherwise, remove it.
CDEFS = -D__PIC$(CPU)__  -DSIM

TARGET = main
PIC24_LIB_SOURCES = common/pic24_clockfreq.c common/pic24_configbits.c common/pic24_uart.c common/pic24_util.c common/pic24_serial.c
SOURCES = $(TARGET).c plat.c $(TARGET)_nat.c $(TARGET)_img.c $(PIC24_LIB_SOURCES)
OBJS = $(SOURCES:.c=.o)

ifeq ($(DEBUG),true)
	CDEFS += -g -D__DEBUG__=1
endif

# Note: using $(abspath .) produces a path with /cygdrive/c/..., which isn't
# recognized by pic30-gcc. Not sure how to work around this. For now, hard
# code the path.
ABS_PATH_WIN = c:/Users/bjones/Documents/svns/python-on-a-chip/src/platform/pic24
CINCS = -I$(ABS_PATH_WIN)/. -I$(ABS_PATH_WIN)/include
# Note: use of -mlarge-data below increases code size, but allows a heap of > 6K.
# A better solution: use an __attribute__((far)) in heap.c on the statement
# static PmHeap_t pmHeap;
# This keeps code size lower, but still allows large heaps.
CFLAGS = -mcpu=$(CPU) -mlarge-code -Wall -O1 -Wdeclaration-after-statement -I../../vm $(CDEFS) $(CINCS)
# A generous stack -- too small will call overflows.
# A bit of heap, in case user C code wants to malloc (can't imagine why, though).
LDFLAGS = -Wl,--script="lkr\p$(CPU)_bootldr.gld",--stack=400,--heap=100,--report-mem

export CFLAGS IPM PM_LIB_FN CC AR LDFLAGS


.PHONY: all clean

all : $(TARGET).hex

$(PM_LIB_PATH) : ../../vm/*.c ../../vm/*.h
	make -C ../../vm

$(TARGET).hex : $(OBJS) $(PM_LIB_PATH)
	$(CC) $(LDFLAGS) -o $(TARGET).cof $(OBJS) $(PM_LIB_PATH)
	$(B2H) $(TARGET).cof

# Generate native code and module images from the python source
$(TARGET)_nat.c $(TARGET)_img.c: $(PM_USR_SOURCES)
	$(PMIMGCREATOR) -c -u -o $(TARGET)_img.c --native-file=$(TARGET)_nat.c $(PM_USR_SOURCES)

clean :
	$(MAKE) -C ../../vm clean
	$(RM) $(TARGET).out $(OBJS) $(TARGET)_img.* $(TARGET)_nat.*