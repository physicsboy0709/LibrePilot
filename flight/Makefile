export FLIGHT_MAKEFILE := TRUE
export FLIGHT_ROOT_DIR := $(realpath $(dir $(lastword $(MAKEFILE_LIST))))
export PIOS            := $(FLIGHT_ROOT_DIR)/pios
export FLIGHTLIB       := $(FLIGHT_ROOT_DIR)/libraries
export OPMODULEDIR     := $(FLIGHT_ROOT_DIR)/modules
export OPUAVOBJ        := $(FLIGHT_ROOT_DIR)/uavobjects
export OPUAVTALK       := $(FLIGHT_ROOT_DIR)/uavtalk
export FLIGHT_OUT_DIR  ?= $(CURDIR)

# Define supported board lists
ALL_BOARDS    := coptercontrol oplinkmini revolution osd revoproto simposix discoveryf4bare gpsplatinum revonano

# Short names of each board (used to display board name in parallel builds)
coptercontrol_short    := 'cc  '
oplinkmini_short       := 'oplm'
revolution_short       := 'revo'
osd_short              := 'osd '
revoproto_short        := 'revp'
revonano_short         := 'revn'
simposix_short         := 'posx'
discoveryf4bare_short  := 'df4b'
gpsplatinum_short      := 'gps9'

# SimPosix only builds on Linux so drop it from the list for
# all other platforms.
ifneq ($(UNAME), Linux)
    ALL_BOARDS := $(filter-out simposix, $(ALL_BOARDS))
endif

# Start out assuming that we'll build fw, bl and bu for all boards
FW_BOARDS  := $(ALL_BOARDS)
BL_BOARDS  := $(ALL_BOARDS)
BU_BOARDS  := $(ALL_BOARDS)
EF_BOARDS  := $(ALL_BOARDS)

# SimPosix doesn't have a BL, BU or EF target so we need to
# filter them out to prevent errors on the all_flight target.
BL_BOARDS  := $(filter-out simposix, $(BL_BOARDS))
BU_BOARDS  := $(filter-out simposix gpsplatinum, $(BU_BOARDS))
EF_BOARDS  := $(filter-out simposix, $(EF_BOARDS))

# Generate the targets for whatever boards are left in each list
FW_TARGETS := $(addprefix fw_, $(FW_BOARDS))
BL_TARGETS := $(addprefix bl_, $(BL_BOARDS))
BU_TARGETS := $(addprefix bu_, $(BU_BOARDS))
EF_TARGETS := $(addprefix ef_, $(EF_BOARDS))

ALL_FLIGHT       := all_fw all_bl all_bu all_ef
ALL_FLIGHT_CLEAN := $(addsuffix _clean,$(ALL_FLIGHT))

.PHONY: all_flight all_flight_clean
all_flight:       $(ALL_FLIGHT)
all_flight_clean: $(ALL_FLIGHT_CLEAN)


# TEMPLATES (used to generate build rules)

# $(1) = Canonical board name all in lower case (e.g. coptercontrol)
# $(2) = Short name for board (e.g cc)
define FW_TEMPLATE
.PHONY: $(1) fw_$(1)
$(1): fw_$(1)_opfw
fw_$(1): fw_$(1)_opfw

fw_$(1)_%: flight_uavobjects
	$(V1) $$(ARM_GCC_VERSION_CHECK_TEMPLATE)
	$(V1) mkdir -p $(FLIGHT_OUT_DIR)/fw_$(1)/dep
	$(V1) cd $(FLIGHT_ROOT_DIR)/targets/boards/$(1)/firmware && \
		$$(MAKE) -r --no-print-directory \
		BUILD_TYPE=fw \
		BOARD_NAME=$(1) \
		BOARD_SHORT_NAME=$(2) \
		TOPDIR=$(FLIGHT_ROOT_DIR)/targets/boards/$(1)/firmware \
		OUTDIR=$(FLIGHT_OUT_DIR)/fw_$(1) \
		TARGET=fw_$(1) \
		$$*

.PHONY: $(1)_clean
$(1)_clean: fw_$(1)_clean
fw_$(1)_clean:
	@echo " CLEAN      $(call toprel, $(FLIGHT_OUT_DIR)/fw_$(1))"
	$(V1) rm -fr $(FLIGHT_OUT_DIR)/fw_$(1)
endef

# $(1) = Canonical board name all in lower case (e.g. coptercontrol)
# $(2) = Short name for board (e.g cc)
define BL_TEMPLATE
.PHONY: bl_$(1)
bl_$(1): bl_$(1)_bin
bl_$(1)_bino: bl_$(1)_bin

bl_$(1)_%:
	$(V1) $$(ARM_GCC_VERSION_CHECK_TEMPLATE)
	$(V1) mkdir -p $(FLIGHT_OUT_DIR)/bl_$(1)/dep
	$(V1) cd $(FLIGHT_ROOT_DIR)/targets/boards/$(1)/bootloader && \
		$$(MAKE) -r --no-print-directory \
		BUILD_TYPE=bl \
		BOARD_NAME=$(1) \
		BOARD_SHORT_NAME=$(2) \
		TOPDIR=$(FLIGHT_ROOT_DIR)/targets/boards/$(1)/bootloader \
		OUTDIR=$(FLIGHT_OUT_DIR)/bl_$(1) \
		TARGET=bl_$(1) \
		$$*

.PHONY: unbrick_$(1)
unbrick_$(1): bl_$(1)_hex
$(if $(filter-out undefined,$(origin UNBRICK_TTY)),
	$(V0) @echo " UNBRICK    $(1) via $$(UNBRICK_TTY)"
	$(V1) $(STM32FLASH_DIR)/stm32flash \
		-w $(FLIGHT_OUT_DIR)/bl_$(1)/bl_$(1).hex \
		-g 0x0 \
		$$(UNBRICK_TTY)
,
	$(V0) @echo
	$(V0) @echo "ERROR: You must specify UNBRICK_TTY=<serial-device> to use for unbricking."
	$(V0) @echo "       eg. $$(MAKE) $$@ UNBRICK_TTY=/dev/ttyUSB0"
)

.PHONY: bl_$(1)_clean
bl_$(1)_clean:
	@echo " CLEAN      $(call toprel, $(FLIGHT_OUT_DIR)/bl_$(1))"
	$(V1) rm -fr $(FLIGHT_OUT_DIR)/bl_$(1)
endef

# $(1) = Canonical board name all in lower case (e.g. coptercontrol)
# $(2) = Short name for board (e.g cc)
define BU_TEMPLATE
.PHONY: bu_$(1)
bu_$(1): bu_$(1)_opfw

bu_$(1)_%: bl_$(1)_bino
	$(V1) mkdir -p $(FLIGHT_OUT_DIR)/bu_$(1)/dep
	$(V1) cd $(FLIGHT_ROOT_DIR)/targets/common/bootloader_updater && \
		$$(MAKE) -r --no-print-directory \
		BUILD_TYPE=bu \
		BOARD_NAME=$(1) \
		BOARD_SHORT_NAME=$(2) \
		TOPDIR=$(FLIGHT_ROOT_DIR)/targets/common/bootloader_updater \
		OUTDIR=$(FLIGHT_OUT_DIR)/bu_$(1) \
		TARGET=bu_$(1) \
		$$*

.PHONY: bu_$(1)_clean
bu_$(1)_clean:
	@echo " CLEAN      $(call toprel, $(FLIGHT_OUT_DIR)/bu_$(1))"
	$(V1) rm -fr $(FLIGHT_OUT_DIR)/bu_$(1)
endef

# $(1) = Canonical board name all in lower case (e.g. coptercontrol)
# $(2) = Short name for board (e.g cc)
define EF_TEMPLATE
.PHONY: ef_$(1)
ef_$(1): ef_$(1)_bin

ef_$(1)_%: bl_$(1)_bin fw_$(1)_opfw
	$(V1) mkdir -p $(FLIGHT_OUT_DIR)/ef_$(1)
	$(V1) cd $(FLIGHT_ROOT_DIR)/targets/common/entire_flash && \
		$$(MAKE) -r --no-print-directory \
		BUILD_TYPE=ef \
		BOARD_NAME=$(1) \
		BOARD_SHORT_NAME=$(2) \
		DFU_CMD="$(DFUUTIL_DIR)/bin/dfu-util" \
		TOPDIR=$(FLIGHT_ROOT_DIR)/targets/common/entire_flash \
		OUTDIR=$(FLIGHT_OUT_DIR)/ef_$(1) \
		TARGET=ef_$(1) \
		$$*

.PHONY: ef_$(1)_clean
ef_$(1)_clean:
	@echo " CLEAN      $(call toprel, $(FLIGHT_OUT_DIR)/ef_$(1))"
	$(V1) rm -fr $(FLIGHT_OUT_DIR)/ef_$(1)
endef

# $(1) = Canonical board name all in lower case (e.g. coptercontrol)
define BOARD_PHONY_TEMPLATE
.PHONY: all_$(1)
all_$(1): $$(filter fw_$(1), $$(FW_TARGETS))
all_$(1): $$(filter bl_$(1), $$(BL_TARGETS))
all_$(1): $$(filter bu_$(1), $$(BU_TARGETS))
all_$(1): $$(filter ef_$(1), $$(EF_TARGETS))

.PHONY: all_$(1)_clean
all_$(1)_clean: $$(addsuffix _clean, $$(filter fw_$(1), $$(FW_TARGETS)))
all_$(1)_clean: $$(addsuffix _clean, $$(filter bl_$(1), $$(BL_TARGETS)))
all_$(1)_clean: $$(addsuffix _clean, $$(filter bu_$(1), $$(BU_TARGETS)))
all_$(1)_clean: $$(addsuffix _clean, $$(filter ef_$(1), $$(EF_TARGETS)))
endef


# Generate flight build rules
 .PHONY: first
first: all_fw

.PHONY: all_fw all_fw_clean
all_fw:        $(addsuffix _opfw,  $(FW_TARGETS))
all_fw_clean:  $(addsuffix _clean, $(FW_TARGETS))

.PHONY: all_bl all_bl_clean
all_bl:        $(addsuffix _bin,   $(BL_TARGETS))
all_bl_clean:  $(addsuffix _clean, $(BL_TARGETS))

.PHONY: all_bu all_bu_clean
all_bu:        $(addsuffix _opfw,  $(BU_TARGETS))
all_bu_clean:  $(addsuffix _clean, $(BU_TARGETS))

.PHONY: all_ef all_ef_clean
all_ef:        $(EF_TARGETS)
all_ef_clean:  $(addsuffix _clean, $(EF_TARGETS))

# Expand the groups of targets for each board
$(foreach board, $(ALL_BOARDS), $(eval $(call BOARD_PHONY_TEMPLATE,$(board))))

# Expand the firmware rules
$(foreach board, $(ALL_BOARDS), $(eval $(call FW_TEMPLATE,$(board),$($(board)_short))))

# Expand the bootloader rules
$(foreach board, $(ALL_BOARDS), $(eval $(call BL_TEMPLATE,$(board),$($(board)_short))))

# Expand the bootloader updater rules
$(foreach board, $(ALL_BOARDS), $(eval $(call BU_TEMPLATE,$(board),$($(board)_short))))

# Expand the entire-flash rules
$(foreach board, $(ALL_BOARDS), $(eval $(call EF_TEMPLATE,$(board),$($(board)_short))))

.PHONY: sim_win32
sim_win32: sim_win32_exe

sim_win32_%: flight_uavobjects
	$(V1) mkdir -p $(FLIGHT_OUT_DIR)/sitl_win32
	$(V1) $(MAKE) --no-print-directory \
		-C $(FLIGHT_ROOT_DIR)/targets/OpenPilot --file=$(FLIGHT_ROOT_DIR)/targets/OpenPilot/Makefile.win32 $*

.PHONY: sim_osx
sim_osx: sim_osx_elf

sim_osx_%: flight_uavobjects
	$(V1) mkdir -p $(FLIGHT_OUT_DIR)/sim_osx
	$(V1) $(MAKE) --no-print-directory \
		-C $(FLIGHT_ROOT_DIR)/targets/SensorTest --file=$(FLIGHT_ROOT_DIR)/targets/SensorTest/Makefile.osx $*

##############################
#
# UAV Objects
#
##############################
UAVOBJGENERATOR ?= $(shell which uavobjgenerator)

UAVOBJ_XML_DIR := $(FLIGHT_ROOT_DIR)/../shared/uavobjectdefinition
export FLIGHT_UAVOBJ_DIR := $(FLIGHT_OUT_DIR)/uavobjects

.PHONY: flight_uavobjects
flight_uavobjects: $(UAVOBJGENERATOR)
	@mkdir -p $(FLIGHT_UAVOBJ_DIR)
	$(V1) cd $(FLIGHT_UAVOBJ_DIR) && \
	    $(UAVOBJGENERATOR) -flight $(UAVOBJ_XML_DIR) $(FLIGHT_ROOT_DIR)/..

