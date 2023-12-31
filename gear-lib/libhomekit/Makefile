###############################################################################
# common
###############################################################################
#ARCH: linux/pi/android/ios/
ARCH		?= linux
CROSS_PREFIX	?=
OUTPUT		?= /usr/local
BUILD_DIR	:= $(shell pwd)/../../build/
ARCH_INC	:= $(BUILD_DIR)/$(ARCH).inc
COLOR_INC	:= $(BUILD_DIR)/color.inc

include $(ARCH_INC)
include $(COLOR_INC)

CC_V		?= $(CC)
CXX_V		?= $(CXX)
LD_V		?= $(LD)
AR_V		?= $(AR)
CP_V		?= $(CP)
RM_V		?= $(RM)

###############################################################################
# target and object
###############################################################################
LIBNAME		= libhomekit
VER_TAG		= $(shell echo ${LIBNAME} | tr 'a-z' 'A-Z')
VER		= $(shell awk '/'"${VER_TAG}_VERSION"'/{print $$3}' ${LIBNAME}.h)
TGT_LIB_H	= $(LIBNAME).h
TGT_LIB_A	= $(LIBNAME).a
TGT_LIB_SO	= $(LIBNAME).so
TGT_LIB_SO_VER	= $(TGT_LIB_SO).${VER}
TGT_UNIT_TEST	= test_$(LIBNAME)

OBJS_LIB	= \
		  src/accessories.o \
		  src/base64.o \
		  src/crypto.o \
		  src/debug.o \
		  src/json.o \
		  src/pairing.o \
		  src/port.o \
		  src/query_params.o \
		  src/storage.o \
		  src/server.o \
		  src/tlv.o \
		  mdns.o \
		  accessory.o \
		  camera_session.o \
		  streaming.o \
		  crypto.o \
		  cJSON.o \
		  http-parser/http_parser.o



OBJS_UNIT_TEST	= test_$(LIBNAME).o

###############################################################################
# cflags and ldflags
###############################################################################
ifeq ($(MODE), release)
CFLAGS  := -O2 -Wall -Werror -fPIC
LTYPE   := release
else
CFLAGS  := -g -Wall -fPIC
LTYPE   := debug
endif
ifeq ($(OUTPUT),/usr/local)
OUTLIBPATH :=/usr/local
else
OUTLIBPATH :=$(OUTPUT)/$(LTYPE)
endif
CFLAGS	+= $($(ARCH)_CFLAGS) \
        -DESP_IDF \
        -DWOLFCRYPT_HAVE_SRP \
        -DWOLFSSL_SHA512 \
        -DWOLFSSL_BASE64_ENCODE \
        -DNO_MD5 \
        -DNO_SHA \
        -DHAVE_HKDF \
        -DHAVE_CHACHA \
        -DHAVE_POLY1305 \
        -DHAVE_ED25519 \
        -DHAVE_CURVE25519 \
        -DNO_SESSION_CACHE \
        -DRSA_LOW_MEM \
        -DGCM_SMALL \
        -DUSE_SLOW_SHA512 \
        -DWOLFCRYPT_ONLY

CFLAGS	+= -I$(OUTPUT)/include/gear-lib -I./include/ -I./

SHARED	:= -shared

LDFLAGS	:= $($(ARCH)_LDFLAGS) -lwolfssl -ljpeg -lx264
LDFLAGS	+= -L$(OUTLIBPATH)/lib/gear-lib -lqueue -lposix -ltime
LDFLAGS	+= -pthread

###############################################################################
# target
###############################################################################
.PHONY : all clean

TGT	:= $(TGT_LIB_A)
TGT	+= $(TGT_LIB_SO)
TGT	+= $(TGT_UNIT_TEST)

OBJS	:= $(OBJS_LIB) $(OBJS_UNIT_TEST)

all: $(TGT)

%.o:%.c
	$(CC_V) -c $(CFLAGS) $< -o $@

$(TGT_LIB_A): $(OBJS_LIB)
	$(AR_V) rcs $@ $^

$(TGT_LIB_SO): $(OBJS_LIB)
	$(CC_V) -o $@ $^ $(SHARED)
	@mv $(TGT_LIB_SO) $(TGT_LIB_SO_VER)
	@ln -sf $(TGT_LIB_SO_VER) $(TGT_LIB_SO)

$(TGT_UNIT_TEST): $(OBJS_UNIT_TEST) $(ANDROID_MAIN_OBJ)
	$(CC_V) -o $@ $^ $(TGT_LIB_A) $(LDFLAGS)

clean:
	$(RM_V) -f $(OBJS)
	$(RM_V) -f $(TGT)
	$(RM_V) -f version.h
	$(RM_V) -f $(TGT_LIB_SO)*
	$(RM_V) -f $(TGT_LIB_SO_VER)

install:
	$(MAKEDIR_OUTPUT)
	@if [ "$(MODE)" = "release" ];then $(STRIP) $(TGT); fi
	$(CP_V) -r $(TGT_LIB_H)  $(OUTPUT)/include/gear-lib
	$(CP_V) -r $(TGT_LIB_A)  $(OUTLIBPATH)/lib/gear-lib
	$(CP_V) -r $(TGT_LIB_SO) $(OUTLIBPATH)/lib/gear-lib
	$(CP_V) -r $(TGT_LIB_SO_VER) $(OUTLIBPATH)/lib/gear-lib

uninstall:
	cd $(OUTPUT)/include/gear-lib && rm -f $(TGT_LIB_H)
	$(RM_V) -f $(OUTLIBPATH)/lib/gear-lib/$(TGT_LIB_A)
	$(RM_V) -f $(OUTLIBPATH)/lib/gear-lib/$(TGT_LIB_SO)
	$(RM_V) -f $(OUTLIBPATH)/lib/gear-lib/$(TGT_LIB_SO_VER)
