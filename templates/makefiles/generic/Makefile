# Target
TARGET = target

# CPU and Clock-frequency settings
# Default: atmega328p with 16MHz (16000000L)
MCU = atmega328p
F_CPU = 16000000L

# Avrdude settings
AVRDUDE_PART = m328p
AVRDUDE_PROG = stk500v1
AVRDUDE_PORT = /dev/ttyUSB0
AVRDUDE_BAUD = 57600

# Do not change anything below this line unless
# you really know what you're doing

# Additional sources
SRC =
CXXSRC =

FORMAT = ihex
MAKEFILE = Makefile

CDEFS = -DF_CPU=$(F_CPU)
CXXDEFS = -DF_CPU=$(F_CPU)
CINCLUDES = -I$(BASEPATH)
CXXINCLUDES = -I$(BASEPATH)

# Compiler + linker flags
# Default setting: Optimize for size (-Os)
CFLAGS = -g $(CINCLUDES) -Os $(CDEFS)
CXXFLAGS = -g $(CXXINCLUDES) -Os $(CDEFS)
LDFLAGS = -lm

# Binary paths (This is the stand-alone version, so we'll
# be using some relative paths). Sorry.
AR = avr-ar
CC = avr-gcc
CXX = avr-g++
NM = avr-nm
OBJDUMP = avr-objdump
OBJCOPY = avr-objcopy
AVRDUDE = avrdude
SIZE = avr-size
RM = rm -f
MV = mv -f

# Objects and lists
OBJ = $(SRC:.c=.o) $(CXXSRC:.cpp=.o) $(ASRC:.S=.o)
LST = $(ASRC:.S=.lst) $(CXXSRC:.cpp=.lst) $(SRC:.c=.lst)

ALL_CFLAGS = -mmcu=$(MCU) -I. $(CFLAGS)
ALL_CXXFLAGS = -mmcu=$(MCU) -I. $(CXXFLAGS)
ALL_ASFLAGS = -mmcu=$(MCU) -I. -x assembler-with-cpp

all: elf hex

elf: $(TARGET).elf
hex: $(TARGET).hex
eep: $(TARGET).eep
sym: $(TARGET).sym
lss: $(TARGET).lss

coff: $(TARGET).elf
	$(COFFCONVERT) -O coff-avr $(TARGET).elf $(TARGET).cof

extcoff: $(TARGET).elf
	$(COFFCONVERT) -O coff-ext-avr $(TARGET).elf $(TARGET).cof

.SUFFIXES: .eep .elf .hex .sym .lss

.elf.hex:
	@echo "Building iHex file."
	@$(OBJCOPY) -O $(FORMAT) -R .eeprom $< $@

.elf.eep:
	@echo "Building eeprom file."
	@$(OBJCOPY) -j .eeprom --set-section-flags=.eeprom="alloc,load" \
	--change-section-lma .eeprom=0 -O $(FORMAT) $< $@

.elf.lss:
	$(OBJDUMP) -h -S $< > $@

.elf.sym:
	$(NM) -n $< > $@

$(TARGET).elf: $(TARGET).o
	@echo "Preparing."
	@$(CC) $(ALL_CFLAGS) -o $@ $(TARGET).c -L. $(LDFLAGS)

.cpp.o:
	@echo "Building C++ Source file $@."
	@$(CXX) -c $(ALL_CXXFLAGS) $< -o $@

.c.o:
	@echo "Building C Source file $@."
	@$(CC) -c $(ALL_CFLAGS) $< -o $@

.c.s:
	@echo "Building Assembler Source file $@."
	@$(CC) -S $(ALL_CFLAGS) $< -o $@

.S.o:
	@echo "Building object."
	@$(CC) -c $(ALL_ASFLAGS) $< -o $@

flash:
	@echo "Spawning avrdude."
	@$(AVRDUDE) -p $(AVRDUDE_PART) -c $(AVRDUDE_PROG) -U flash:w:$(TARGET).hex -P $(AVRDUDE_PORT) -b $(AVRDUDE_BAUD)
	@echo "All done. Have fun with your project."

clean:
	@echo "Cleaning up."
	@$(RM) $(TARGET).hex $(TARGET).eep $(TARGET).cof \
	@$(TARGET).map $(TARGET).elf $(TARGET).sym \
	@$(TARGET).lss $(TARGET).o \
	@$(OBJ) $(LST) $(SRC:.c=.s) $(SRC:.c=.d) $(CXXSRC:.cpp=.s) $(CXXSRC:.cpp=.d)

depend:
	if grep '^# Mark - do not delete' $(MAKEFILE) > /dev/null
	then \
		sed -e '/^# Mark - do not delete/,$$d' $(MAKEFILE) > \
		$(MAKEFILE).$$$$ && \
		$(MV) $(MAKEFILE).$$$$ $(MAKEFILE); \
	fi
	echo "# Mark - do not delete" >> $(MAKEFILE); \
	$(CC) -M -mmcu=$(MCU) $(CDEFS) $(CINCS) $(SRC) $(ASRC) >> $(MAKEFILE)

# Mark - do not delete
