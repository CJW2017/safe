# Safe: Encrypted File System
# Copyright (C) 2014 Rian Hunter <rian@alum.mit.edu>
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU Lesser General Public License as published by
# the Free Software Foundation, either version 2 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU Lesser General Public License for more details.
#
# You should have received a copy of the GNU Lesser General Public License
# along with this program.  If not, see <http://www.gnu.org/licenses/>

include config.mk

DLLTOOL ?= dlltool
STRIP ?= strip

SAFE_RAMDISK_SRC_CPP = ramdisk.cpp nt_helpers.cpp ramdisk_device.cpp tfs_dav_reparse_engage.cpp io_device.cpp ramdisk_control_device.cpp
SAFE_RAMDISK_OBJ =${SAFE_RAMDISK_SRC_CPP:.cpp=.cpp.o}

CPPFLAGS ?= -I$(CURDIR) $(DDKINCLUDE) -I$(CURDIR)/../../src $(if $(RELEASE),,-DDBG)
CXXFLAGS ?= $(CPPFLAGS) -fno-rtti -fno-exceptions -nostdinc++ -fno-threadsafe-statics -std=c++11 -Wall -Werror $(if $(RELEASE),-Os,)
CFLAGS ?= $(CPPFLAGS) -Wall -Werror $(if $(RELEASE),-Os,)

ifdef IS_64BIT_BUILD
EXTRA_LDFLAGS = -Wl,--entry,DriverEntry
SAFE_RAMDISK_TARGET = safe_ramdisk_x64.sys

else
USE_CUSTOM_NTOSKERNEL_DEF = YES
EXTRA_LDFLAGS = -Wl,--image-base,0x10000 \
		-Wl,--file-alignment,0x1000 \
		-Wl,--section-alignment,0x1000 \
		-Wl,--entry,_DriverEntry@8
SAFE_RAMDISK_TARGET = safe_ramdisk.sys
endif

all: $(SAFE_RAMDISK_TARGET)

*.o: *.hpp

%.cpp.o: %.cpp
	$(CXX) $(CXXFLAGS) -c -o $@ $<

%.c.o: %.c
	$(CC) $(CFLAGS) -c -o $@ $<

%.a: %.def
	$(DLLTOOL) -k -d $^ -l $@

$(SAFE_RAMDISK_TARGET): $(SAFE_RAMDISK_OBJ) $(if $(USE_CUSTOM_NTOSKERNEL_DEF),libmyntoskrnl.a,)
	$(CC)	-Wl,--subsystem,native \
                -Wl,--disable-runtime-pseudo-reloc \
                -Wl,--wdmdriver  \
		$(EXTRA_LDFLAGS) \
	        -shared \
		-nostartfiles -nostdlib -static-libgcc \
		-o $@ \
		$(CFLAGS) \
		-L$(CURDIR) \
		$(SAFE_RAMDISK_OBJ) \
		$(if $(USE_CUSTOM_NTOSKERNEL_DEF),-lmyntoskrnl,-lntoskrnl) -lhal -lgcc
	$(if $(RELEASE),$(STRIP) -s $@,)
# TODO: use stampinf or similar tool to make this automatic
	@echo "New safe_ramdisk.sys made, make sure to tweak DriverVer in the safe_ramdisk.inf"

elevate.exe: elevate.c
	$(CC) -mconsole -Wall -Werror -o $@ $^

install_non_pnp_device.exe: install_non_pnp_device.c
	$(CC) -mconsole -Wall -Werror -std=c99 -o $@ $^ -lsetupapi -lnewdev

reload: $(SAFE_SIMREP_TARGET) elevate.exe \
	install_non_pnp_device.exe $(SAFE_RAMDISK_TARGET)
	elevate.exe //wait reload.bat

clean:
	rm -f $(SAFE_SIMREP_TARGET) $(SAFE_RAMDISK_TARGET)
	rm -f $(SAFE_SIMREP_OBJ) $(SAFE_RAMDISK_OBJ)
	rm -f libmyntoskrnl.a libmyfltmrg.a
	rm -f elevate.exe install_non_pnp_device.exe

.PHONY: clean install
