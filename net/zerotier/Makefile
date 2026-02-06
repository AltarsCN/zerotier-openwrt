#
# Copyright (C) 2015-2024 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=zerotier
PKG_VERSION:=1.16.0
PKG_RELEASE:=1

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=https://codeload.github.com/zerotier/ZeroTierOne/tar.gz/$(PKG_VERSION)?
PKG_HASH:=aa9de313d365bf0efb3871aaa56f2d323a08f46df47b627c4eff4f4203fa7fc5
PKG_BUILD_DIR:=$(BUILD_DIR)/ZeroTierOne-$(PKG_VERSION)

PKG_MAINTAINER:=Your Name <your@email.com>
PKG_LICENSE:=MPL-2.0
PKG_LICENSE_FILES:=LICENSE.txt

PKG_ASLR_PIE:=0
PKG_BUILD_PARALLEL:=1
PKG_BUILD_FLAGS:=gc-sections

include $(INCLUDE_DIR)/package.mk

define Package/zerotier
  SECTION:=net
  CATEGORY:=Network
  DEPENDS:=+libpthread +libstdcpp +kmod-tun +ip +libminiupnpc +libnatpmp +libatomic
  TITLE:=ZeroTier One with Controller Support
  URL:=https://www.zerotier.com
  SUBMENU:=VPN
  PROVIDES:=zerotier
endef

define Package/zerotier/description
  ZeroTier creates a global provider-independent virtual private cloud network.
  This package includes built-in network controller support (ZT_CONTROLLER=1).
  The embedded controller uses file-based storage (FileDB) and does not require
  PostgreSQL, Redis, or Rust. Network/member configs are stored as JSON files.
endef

define Package/zerotier/config
	source "$(SOURCE)/Config.in"
endef

# Make binary smaller
TARGET_CFLAGS += -Wl,-z,noexecstack
TARGET_LDFLAGS += -Wl,--as-needed -Wl,-z,noexecstack

# Prevent `-isystem ext` from causing wrong miniupnpc header
TARGET_CFLAGS += -isystem $(STAGING_DIR)/usr/include

ifeq ($(CONFIG_ZEROTIER_ENABLE_DEBUG),y)
MAKE_FLAGS += ZT_DEBUG=1
endif

MAKE_FLAGS += \
	ZT_EMBEDDED=1 \
	ZT_SSO_SUPPORTED=0 \
	DEFS="" \
	OSTYPE="Linux"

# Controller support: ZT_NONFREE=1 is required to compile nonfree/controller/ objects.
# Without ZT_CONTROLLER_USE_LIBPQ (removed by Build/Prepare patch), the controller
# uses FileDB (file-based JSON storage) instead of PostgreSQL/Redis.
ifeq ($(CONFIG_ZEROTIER_ENABLE_CONTROLLER),y)
MAKE_FLAGS += ZT_CONTROLLER=1 ZT_NONFREE=1
else
MAKE_FLAGS += ZT_NONFREE=0
endif

define Build/Prepare
	$(call Build/Prepare/Default)
ifeq ($(CONFIG_ZEROTIER_ENABLE_CONTROLLER),y)
	@echo ">>> Patching make-linux.mk for embedded controller (FileDB, no PQ/Redis/SMEE)"
	# 1. Remove ZT_CONTROLLER_USE_LIBPQ define (keeps -DZT_NO_PEER_METRICS etc.)
	#    Without this define, CV1/CV2/CtlUtil compile to empty .o (all code is #ifdef-guarded).
	#    EmbeddedNetworkController falls back to FileDB as storage backend.
	sed -i 's/ *-DZT_CONTROLLER_USE_LIBPQ//' $(PKG_BUILD_DIR)/make-linux.mk
	# 2. Remove ZT_OPENTELEMETRY_ENABLED (needs full OTEL SDK with exporters/gRPC,
	#    but embedded builds only have the api-only headers)
	sed -i 's/ *-DZT_OPENTELEMETRY_ENABLED[^ ]*//' $(PKG_BUILD_DIR)/make-linux.mk
	# 3. Remove libpqxx/hiredis/redis library linkage (unique pattern on these lines)
	sed -i '/libpqxx-7\.7\.3/d' $(PKG_BUILD_DIR)/make-linux.mk
	# 4. Remove PostgreSQL include path
	sed -i '/\/usr\/include\/postgresql/d' $(PKG_BUILD_DIR)/make-linux.mk
	# 5. Remove smeeclient.a library linking (both debug and release)
	sed -i '/libsmeeclient\.a/d' $(PKG_BUILD_DIR)/make-linux.mk
	# 6. Replace smeeclient Rust/cargo build with no-op (embedded doesn't need Rust)
	sed -i 's|export PATH.*cargo build.*smeeclient|@echo "Skipping smeeclient for embedded build"|' $(PKG_BUILD_DIR)/make-linux.mk
	@echo ">>> Patch complete. Embedded controller uses FileDB (JSON file storage)."
endif
endef

define Build/Compile
	$(call Build/Compile/Default,one)
ifeq ($(CONFIG_ZEROTIER_ENABLE_SELFTEST),y)
	$(call Build/Compile/Default,selftest)
endif
endef

define Package/zerotier/conffiles
/etc/config/zerotier
/etc/zerotier.conf
/etc/zerotier
endef

define Package/zerotier/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/zerotier-one $(1)/usr/bin/
	$(LN) zerotier-one $(1)/usr/bin/zerotier-cli
	$(LN) zerotier-one $(1)/usr/bin/zerotier-idtool
	$(INSTALL_DIR) $(1)/etc/uci-defaults
ifeq ($(CONFIG_ZEROTIER_ENABLE_SELFTEST),y)
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/zerotier-selftest $(1)/usr/bin/
endif
	$(CP) ./files/* $(1)/
endef

$(eval $(call BuildPackage,zerotier))
