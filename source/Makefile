# GNU Make required
#
# Tested platforms: Linux x86, Linux x86_64, MinGW x86
#

PACKAGE=aq2-tng
VERSION=2.82
FILE_VERSION:=$(VERSION)

## Export env TNG_BUILD_FOR=WIN32 or TNG_BUILD_FOR=WIN64 to build 32-bit exe on 64-bit arch
ifeq ($(TNG_BUILD_FOR),WIN32)
    WIN32=yes
endif

ifeq ($(TNG_BUILD_FOR),WIN64)
    WIN64=yes
endif

ifndef RELEASE
GIT_DATE=$(shell git show -s --format='%cd' --date=short)
ifeq ($(GIT_DATE),)
	GIT_DATE=$(shell date +'%Y-%m-%d')
endif
VERSION += $(GIT_DATE)
GIT_REV=$(shell git rev-parse --short HEAD)
ifeq ($(GIT_REV),)
	GIT_REV=git
endif
VERSION += $(GIT_REV)
FILE_VERSION:=$(FILE_VERSION)~$(GIT_REV)
endif

CC?=gcc
BASE_CFLAGS=-Wall -fno-strict-aliasing -DC_ONLY -DVERSION="\"$(VERSION)\""

ifdef WEXTRA
BASE_CFLAGS += -Wextra
else
# Temporary fix to hide a few warnings that haven't been fixed yet.
BASE_CFLAGS += -Wno-format-truncation -Wno-unused-result
endif

#use these cflags to optimize it
ifdef DEBUG
	CFLAGS=$(BASE_CFLAGS) -g
else
	CFLAGS=$(BASE_CFLAGS) -O2
endif

ifdef NATIVE
# Allow machine-specific optimizations.
# This is no longer default because some build environments don't like it.
CFLAGS+=-march=native -mtune=native
endif

MACHINE=$(shell uname -m)
SYSTEM=$(shell uname -s)

# Fixes by bAron to get it compiling on max m1
ifeq ($(TNG_BUILD_FOR),M1)
	CC=clang
	MACHINE=arm64e
endif

ARCH=$(MACHINE)

# If machine starts with arm (any 32-bit Raspberry Pi) make gamearm.so.
ifneq (,$(findstring __arm,__$(MACHINE)))
ARCH=arm
endif

DO_CC=$(CC) $(CFLAGS) $(SHLIBCFLAGS) -o $@ -c $<
DO_LINK=$(CC) $(CFLAGS) $(SHLIBLDFLAGS) -o $@ $(GAME_OBJS) $(SHLIBLIBS)

SHLIBEXT?=so
SHLIBLDFLAGS?=-shared

ifneq (,$(findstring MINGW,$(SYSTEM))$(findstring CYGWIN,$(SYSTEM)))
WIN32=yes
endif

ifdef WIN32
ifneq (,$(findstring Linux,$(SYSTEM)))
CC=i686-w64-mingw32-gcc
endif
CFLAGS+=-DWIN32
ARCH=x86
SHLIBEXT=dll
SHLIBLIBS=-lws2_32
else
CFLAGS+=-fPIC
endif

ifdef WIN64
CC=x86_64-w64-mingw32-gcc
CFLAGS+=-DWIN64 -m64
ARCH=x86_64
SHLIBEXT=dll
SHLIBLIBS=-lws2_32
endif

ifeq ($(ARCH),x86)
CFLAGS+=-m32
endif


ifdef NO_BOTS
CFLAGS+=-DNO_BOTS
endif


#############################################################################
# SETUP AND BUILD
# GAME
#############################################################################

%.o: %.c $(wildcard *.h */*.h)
	$(DO_CC)

GAME_OBJS = \
	a_cmds.o a_ctf.o a_doorkick.o a_game.o a_items.o a_match.o \
	a_menu.o a_radio.o a_team.o a_tourney.o a_vote.o a_xcmds.o a_xgame.o \
	a_xmenu.o a_xvote.o cgf_sfx_glass.o g_chase.o g_cmds.o \
	g_combat.o g_func.o g_items.o g_main.o g_misc.o \
	g_phys.o g_save.o g_spawn.o g_svcmds.o g_target.o g_trigger.o \
	g_utils.o g_weapon.o \
	p_antilag.o p_client.o p_hud.o p_view.o p_weapon.o q_shared.o \
	tng_stats.o tng_flashlight.o tng_irc.o tng_ini.o tng_balancer.o \
	g_grapple.o a_dom.o tng_jump.o

ifndef NO_BOTS
GAME_OBJS += acesrc/acebot_ai.o acesrc/acebot_nodes.o acesrc/bot_combat.o \
	acesrc/bot_states.o acesrc/acebot_cmds.o acesrc/acebot_spawn.o \
	acesrc/bot_env_doors.o acesrc/bot_utility.o acesrc/acebot_compress.o \
	acesrc/bot_ai.o acesrc/bot_movement.o acesrc/bot_weapon.o \
	acesrc/acebot_items.o acesrc/botchat.o acesrc/botnav.o \
	acesrc/acebot_movement.o acesrc/bot_collision.o acesrc/botscan.o
endif

game$(ARCH).$(SHLIBEXT): $(GAME_OBJS)
	$(DO_LINK)


#############################################################################
# MISC
#############################################################################

clean:
	-rm -f $(GAME_OBJS) game$(ARCH).$(SHLIBEXT)

install:
	cp game$(ARCH).$(SHLIBEXT) ../action/
	strip ../action/game$(ARCH).$(SHLIBEXT)

bindist: game$(ARCH).$(SHLIBEXT)
	strip -s game$(ARCH).$(SHLIBEXT)
	tar -zcvf ../$(PACKAGE)-$(FILE_VERSION)-$(SYSTEM)-$(MACHINE).tar.gz \
            game$(ARCH).$(SHLIBEXT) ../TNG-manual.txt ../change.txt

distclean: clean
	-rm -f ../$(PACKAGE)-*.{tar.gz,zip}

dist:
	git archive --format=tar --prefix=$(PACKAGE)-$(FILE_VERSION)-src/ HEAD | gzip -9 > ../$(PACKAGE)-$(FILE_VERSION)-src.tar.gz

