# SPDX-License-Identifier: GPL-2.0-only
#
# Copyright (C) 2006-2021 OpenWrt.org

include $(TOPDIR)/rules.mk

PKG_NAME:=hostapd
PKG_RELEASE:=1

PKG_SOURCE_URL:=https://w1.fi/hostap.git
PKG_SOURCE_PROTO:=git
PKG_SOURCE_DATE:=2025-06-27
PKG_SOURCE_VERSION:=ea08700a762f13c415e319993720bed83ad9aa2c
PKG_MIRROR_HASH:=8bfd23188058611a7efb3b0064a7601f8073ceedbf1538a99f93abc743608d46

PKG_MAINTAINER:=Felix Fietkau <nbd@nbd.name>
PKG_LICENSE:=BSD-3-Clause
PKG_CPE_ID:=cpe:/a:w1.fi:hostapd

PKG_BUILD_PARALLEL:=1
PKG_ASLR_PIE_REGULAR:=1

PKG_CONFIG_DEPENDS:= \
	CONFIG_PACKAGE_hostapd \
	CONFIG_PACKAGE_hostapd-basic \
	CONFIG_PACKAGE_hostapd-mini \
	CONFIG_WPA_RFKILL_SUPPORT \
	CONFIG_DRIVER_11AC_SUPPORT \
	CONFIG_DRIVER_11AX_SUPPORT \
	CONFIG_DRIVER_11BE_SUPPORT \
	CONFIG_WPA_ENABLE_WEP

PKG_BUILD_FLAGS:=gc-sections lto

EAPOL_TEST_PROVIDERS:=eapol-test eapol-test-openssl eapol-test-wolfssl

SUPPLICANT_PROVIDERS:=
HOSTAPD_PROVIDERS:=

LOCAL_TYPE=$(strip \
		$(if $(findstring wpad,$(BUILD_VARIANT)),wpad, \
		$(if $(findstring supplicant,$(BUILD_VARIANT)),supplicant, \
		hostapd \
		)))

LOCAL_AND_LIB_VARIANT=$(patsubst hostapd-%,%,\
		      $(patsubst wpad-%,%,\
		      $(patsubst supplicant-%,%,\
		      $(BUILD_VARIANT)\
		      )))

LOCAL_VARIANT=$(patsubst %-internal,%,\
	      $(patsubst %-openssl,%,\
	      $(patsubst %-wolfssl,%,\
	      $(patsubst %-mbedtls,%,\
	      $(LOCAL_AND_LIB_VARIANT)\
	      ))))

SSL_VARIANT=$(strip \
		$(if $(findstring openssl,$(LOCAL_AND_LIB_VARIANT)),openssl,\
		$(if $(findstring wolfssl,$(LOCAL_AND_LIB_VARIANT)),wolfssl,\
		$(if $(findstring mbedtls,$(LOCAL_AND_LIB_VARIANT)),mbedtls,\
		internal\
		))))

CONFIG_VARIANT:=$(LOCAL_VARIANT)
ifeq ($(LOCAL_VARIANT),mesh)
  CONFIG_VARIANT:=full
endif

include $(INCLUDE_DIR)/package.mk

STAMP_CONFIGURED:=$(STAMP_CONFIGURED)_$(CONFIG_WPA_MSG_MIN_PRIORITY)

ifneq ($(CONFIG_DRIVER_11AC_SUPPORT),)
  HOSTAPD_IEEE80211AC:=y
endif

ifneq ($(CONFIG_DRIVER_11AX_SUPPORT),)
  HOSTAPD_IEEE80211AX:=y
endif

ifneq ($(LOCAL_VARIANT),mini)
  ifneq ($(CONFIG_DRIVER_11BE_SUPPORT),)
    HOSTAPD_IEEE80211BE:=y
  endif
endif


CORE_DEPENDS = +ucode +libucode \
	+ucode-mod-fs +ucode-mod-nl80211 +ucode-mod-rtnl +ucode-mod-ubus +ucode-mod-uloop \
	+libubus +libblobmsg-json \
	+libudebug
OPENSSL_DEPENDS = +PACKAGE_$(1):libopenssl +PACKAGE_$(1):libopenssl-legacy

DRIVER_MAKEOPTS= \
	CONFIG_ACS=y CONFIG_DRIVER_NL80211=y \
	CONFIG_IEEE80211AC=$(HOSTAPD_IEEE80211AC) \
	CONFIG_IEEE80211AX=$(HOSTAPD_IEEE80211AX) \
	CONFIG_IEEE80211BE=$(HOSTAPD_IEEE80211BE) \
	CONFIG_MBO=$(CONFIG_WPA_MBO_SUPPORT) \
	CONFIG_UCODE=y CONFIG_APUP=y \
	$(if $(HOSTAPD_IEEE80211BE),CONFIG_OCV=y)


ifeq ($(SSL_VARIANT),openssl)
  DRIVER_MAKEOPTS += CONFIG_TLS=openssl CONFIG_SAE=y
  TARGET_LDFLAGS += -lcrypto -lssl

  ifeq ($(LOCAL_VARIANT),basic)
    DRIVER_MAKEOPTS += CONFIG_OWE=y
  endif
  ifeq ($(LOCAL_VARIANT),mesh)
    DRIVER_MAKEOPTS += CONFIG_AP=y CONFIG_MESH=y
  endif
  ifeq ($(LOCAL_VARIANT),full)
    DRIVER_MAKEOPTS += CONFIG_OWE=y CONFIG_SUITEB192=y CONFIG_AP=y CONFIG_MESH=y CONFIG_EAP_PWD=y CONFIG_DPP=y CONFIG_DPP2=y
  endif
endif

ifeq ($(SSL_VARIANT),wolfssl)
  DRIVER_MAKEOPTS += CONFIG_TLS=wolfssl CONFIG_SAE=y
  TARGET_LDFLAGS += -lwolfssl

  ifeq ($(LOCAL_VARIANT),basic)
    DRIVER_MAKEOPTS += CONFIG_OWE=y
  endif
  ifeq ($(LOCAL_VARIANT),mesh)
    DRIVER_MAKEOPTS += CONFIG_AP=y CONFIG_MESH=y CONFIG_WPS_NFC=1
  endif
  ifeq ($(LOCAL_VARIANT),full)
    DRIVER_MAKEOPTS += CONFIG_OWE=y CONFIG_SUITEB192=y CONFIG_AP=y CONFIG_MESH=y CONFIG_WPS_NFC=1 CONFIG_EAP_PWD=y
  endif
endif

ifeq ($(SSL_VARIANT),mbedtls)
  DRIVER_MAKEOPTS += CONFIG_TLS=mbedtls CONFIG_SAE=y
  TARGET_LDFLAGS += -lmbedcrypto -lmbedx509 -lmbedtls

  ifeq ($(LOCAL_VARIANT),basic)
    DRIVER_MAKEOPTS += CONFIG_OWE=y
  endif
  ifeq ($(LOCAL_VARIANT),mesh)
    DRIVER_MAKEOPTS += CONFIG_AP=y CONFIG_MESH=y CONFIG_WPS_NFC=1
  endif
  ifeq ($(LOCAL_VARIANT),full)
    DRIVER_MAKEOPTS += CONFIG_OWE=y CONFIG_SUITEB192=y CONFIG_AP=y CONFIG_MESH=y CONFIG_WPS_NFC=1 CONFIG_EAP_PWD=y CONFIG_DPP=y CONFIG_DPP2=y
  endif
endif

ifneq ($(LOCAL_TYPE),hostapd)
  ifdef CONFIG_WPA_RFKILL_SUPPORT
    DRIVER_MAKEOPTS += NEED_RFKILL=y
  endif
endif

DRV_DEPENDS:=+libnl-tiny


define Package/hostapd/Default
  SECTION:=net
  CATEGORY:=Network
  SUBMENU:=WirelessAPD
  TITLE:=IEEE 802.1x Authenticator
  URL:=https://w1.fi/hostapd/
  DEPENDS:=$(DRV_DEPENDS) +hostapd-common $(CORE_DEPENDS)
  EXTRA_DEPENDS:=hostapd-common (=$(PKG_VERSION)-r$(PKG_RELEASE))
  USERID:=network=101:network=101
  PROVIDES:=hostapd
  CONFLICTS:=$(HOSTAPD_PROVIDERS)
  HOSTAPD_PROVIDERS+=$(1)
endef

define Package/hostapd
$(call Package/hostapd/Default,$(1))
  TITLE+= (built-in full)
  VARIANT:=full-internal
endef

define Package/hostapd/description
 This package contains a full featured IEEE 802.1x/WPA/EAP/RADIUS
 Authenticator.
endef

define Package/hostapd-openssl
$(call Package/hostapd/Default,$(1))
  TITLE+= (OpenSSL full)
  VARIANT:=full-openssl
  DEPENDS+=$(OPENSSL_DEPENDS)
endef

Package/hostapd-openssl/description = $(Package/hostapd/description)

define Package/hostapd-wolfssl
$(call Package/hostapd/Default,$(1))
  TITLE+= (wolfSSL full)
  VARIANT:=full-wolfssl
  DEPENDS+=+PACKAGE_hostapd-wolfssl:libwolfssl
endef

Package/hostapd-wolfssl/description = $(Package/hostapd/description)

define Package/hostapd-mbedtls
$(call Package/hostapd/Default,$(1))
  TITLE+= (mbedTLS full)
  VARIANT:=full-mbedtls
  DEPENDS+=+PACKAGE_hostapd-mbedtls:libmbedtls
endef

Package/hostapd-mbedtls/description = $(Package/hostapd/description)

define Package/hostapd-basic
$(call Package/hostapd/Default,$(1))
  TITLE+= (WPA-PSK, 11r, 11w)
  VARIANT:=basic
endef

define Package/hostapd-basic/description
 This package contains a basic IEEE 802.1x/WPA Authenticator with WPA-PSK, 802.11r and 802.11w support.
endef

define Package/hostapd-basic-openssl
$(call Package/hostapd/Default,$(1))
  TITLE+= (WPA-PSK, 11r and 11w)
  VARIANT:=basic-openssl
  DEPENDS+=+PACKAGE_hostapd-basic-openssl:libopenssl
endef

define Package/hostapd-basic-openssl/description
 This package contains a basic IEEE 802.1x/WPA Authenticator with WPA-PSK, 802.11r and 802.11w support.
endef

define Package/hostapd-basic-wolfssl
$(call Package/hostapd/Default,$(1))
  TITLE+= (WPA-PSK, 11r and 11w)
  VARIANT:=basic-wolfssl
  DEPENDS+=+PACKAGE_hostapd-basic-wolfssl:libwolfssl
endef

define Package/hostapd-basic-wolfssl/description
 This package contains a basic IEEE 802.1x/WPA Authenticator with WPA-PSK, 802.11r and 802.11w support.
endef

define Package/hostapd-basic-mbedtls
$(call Package/hostapd/Default,$(1))
  TITLE+= (WPA-PSK, 11r and 11w)
  VARIANT:=basic-mbedtls
  DEPENDS+=+PACKAGE_hostapd-basic-mbedtls:libmbedtls
endef

define Package/hostapd-basic-mbedtls/description
 This package contains a basic IEEE 802.1x/WPA Authenticator with WPA-PSK, 802.11r and 802.11w support.
endef

define Package/hostapd-mini
$(call Package/hostapd/Default,$(1))
  TITLE+= (WPA-PSK only)
  VARIANT:=mini
endef

define Package/hostapd-mini/description
 This package contains a minimal IEEE 802.1x/WPA Authenticator (WPA-PSK only).
endef


define Package/wpad/Default
  SECTION:=net
  CATEGORY:=Network
  SUBMENU:=WirelessAPD
  TITLE:=IEEE 802.1x Auth/Supplicant
  DEPENDS:=$(DRV_DEPENDS) +hostapd-common $(CORE_DEPENDS)
  EXTRA_DEPENDS:=hostapd-common (=$(PKG_VERSION)-r$(PKG_RELEASE))
  USERID:=network=101:network=101
  URL:=https://w1.fi/
  PROVIDES:=hostapd wpa-supplicant
  CONFLICTS:=$(HOSTAPD_PROVIDERS) $(SUPPLICANT_PROVIDERS)
  HOSTAPD_PROVIDERS+=$(1)
  SUPPLICANT_PROVIDERS+=$(1)
endef

define Package/wpad
$(call Package/wpad/Default,$(1))
  TITLE+= (built-in full)
  VARIANT:=wpad-full-internal
endef

define Package/wpad/description
 This package contains a full featured IEEE 802.1x/WPA/EAP/RADIUS
 Authenticator and Supplicant
endef

define Package/wpad-openssl
$(call Package/wpad/Default,$(1))
  TITLE+= (OpenSSL full)
  VARIANT:=wpad-full-openssl
  DEPENDS+=$(OPENSSL_DEPENDS)
endef

Package/wpad-openssl/description = $(Package/wpad/description)

define Package/wpad-wolfssl
$(call Package/wpad/Default,$(1))
  TITLE+= (wolfSSL full)
  VARIANT:=wpad-full-wolfssl
  DEPENDS+=+PACKAGE_wpad-wolfssl:libwolfssl
endef

Package/wpad-wolfssl/description = $(Package/wpad/description)

define Package/wpad-mbedtls
$(call Package/wpad/Default,$(1))
  TITLE+= (mbedTLS full)
  VARIANT:=wpad-full-mbedtls
  DEPENDS+=+PACKAGE_wpad-mbedtls:libmbedtls
endef

Package/wpad-mbedtls/description = $(Package/wpad/description)

define Package/wpad-basic
$(call Package/wpad/Default,$(1))
  TITLE+= (WPA-PSK, 11r, 11w)
  VARIANT:=wpad-basic
endef

define Package/wpad-basic/description
 This package contains a basic IEEE 802.1x/WPA Authenticator and Supplicant with WPA-PSK, 802.11r and 802.11w support.
endef

define Package/wpad-basic-openssl
$(call Package/wpad/Default,$(1))
  TITLE+= (OpenSSL, 11r, 11w)
  VARIANT:=wpad-basic-openssl
  DEPENDS+=$(OPENSSL_DEPENDS)
endef

define Package/wpad-basic-openssl/description
 This package contains a basic IEEE 802.1x/WPA Authenticator and Supplicant with WPA-PSK, SAE (WPA3-Personal), 802.11r and 802.11w support.
endef

define Package/wpad-basic-wolfssl
$(call Package/wpad/Default,$(1))
  TITLE+= (wolfSSL, 11r, 11w)
  VARIANT:=wpad-basic-wolfssl
  DEPENDS+=+PACKAGE_wpad-basic-wolfssl:libwolfssl
endef

define Package/wpad-basic-wolfssl/description
 This package contains a basic IEEE 802.1x/WPA Authenticator and Supplicant with WPA-PSK, SAE (WPA3-Personal), 802.11r and 802.11w support.
endef

define Package/wpad-basic-mbedtls
$(call Package/wpad/Default,$(1))
  TITLE+= (mbedTLS, 11r, 11w)
  VARIANT:=wpad-basic-mbedtls
  DEPENDS+=+PACKAGE_wpad-basic-mbedtls:libmbedtls
endef

define Package/wpad-basic-mbedtls/description
 This package contains a basic IEEE 802.1x/WPA Authenticator and Supplicant with WPA-PSK, SAE (WPA3-Personal), 802.11r and 802.11w support.
endef

define Package/wpad-mini
$(call Package/wpad/Default,$(1))
  TITLE+= (WPA-PSK only)
  VARIANT:=wpad-mini
endef

define Package/wpad-mini/description
 This package contains a minimal IEEE 802.1x/WPA Authenticator and Supplicant (WPA-PSK only).
endef

define Package/wpad-mesh
$(call Package/wpad/Default,$(1))
  DEPENDS+=@(!TARGET_uml||BROKEN)
  PROVIDES+=wpa-supplicant-mesh wpad-mesh
endef

define Package/wpad-mesh/description
 This package contains a minimal IEEE 802.1x/WPA Authenticator and Supplicant (with 802.11s mesh and SAE support).
endef

define Package/wpad-mesh-openssl
$(call Package/wpad-mesh,$(1))
  TITLE+= (OpenSSL, 11s, SAE)
  DEPENDS+=$(OPENSSL_DEPENDS)
  VARIANT:=wpad-mesh-openssl
endef

Package/wpad-mesh-openssl/description = $(Package/wpad-mesh/description)

define Package/wpad-mesh-wolfssl
$(call Package/wpad-mesh,$(1))
  TITLE+= (wolfSSL, 11s, SAE)
  DEPENDS+=+PACKAGE_wpad-mesh-wolfssl:libwolfssl
  VARIANT:=wpad-mesh-wolfssl
endef

Package/wpad-mesh-wolfssl/description = $(Package/wpad-mesh/description)

define Package/wpad-mesh-mbedtls
$(call Package/wpad-mesh,$(1))
  TITLE+= (mbedTLS, 11s, SAE)
  DEPENDS+=+PACKAGE_wpad-mesh-mbedtls:libmbedtls
  VARIANT:=wpad-mesh-mbedtls
endef

Package/wpad-mesh-mbedtls/description = $(Package/wpad-mesh/description)


define Package/wpa-supplicant/Default
  SECTION:=net
  CATEGORY:=Network
  SUBMENU:=WirelessAPD
  TITLE:=WPA Supplicant
  URL:=https://w1.fi/wpa_supplicant/
  DEPENDS:=$(DRV_DEPENDS) +hostapd-common $(CORE_DEPENDS)
  EXTRA_DEPENDS:=hostapd-common (=$(PKG_VERSION)-r$(PKG_RELEASE))
  USERID:=network=101:network=101
  PROVIDES:=wpa-supplicant
  CONFLICTS:=$(SUPPLICANT_PROVIDERS)
  SUPPLICANT_PROVIDERS+=$(1)
endef

define Package/wpa-supplicant
$(call Package/wpa-supplicant/Default,$(1))
  TITLE+= (built-in full)
  VARIANT:=supplicant-full-internal
endef

define Package/wpa-supplicant-openssl
$(call Package/wpa-supplicant/Default,$(1))
  TITLE+= (OpenSSL full)
  VARIANT:=supplicant-full-openssl
  DEPENDS+=$(OPENSSL_DEPENDS)
endef

define Package/wpa-supplicant-wolfssl
$(call Package/wpa-supplicant/Default,$(1))
  TITLE+= (wolfSSL full)
  VARIANT:=supplicant-full-wolfssl
  DEPENDS+=+PACKAGE_wpa-supplicant-wolfssl:libwolfssl
endef

define Package/wpa-supplicant-mbedtls
$(call Package/wpa-supplicant/Default,$(1))
  TITLE+= (mbedTLS full)
  VARIANT:=supplicant-full-mbedtls
  DEPENDS+=+PACKAGE_wpa-supplicant-mbedtls:libmbedtls
endef

define Package/wpa-supplicant/config
	source "$(SOURCE)/Config.in"
endef

define Package/wpa-supplicant-p2p
$(call Package/wpa-supplicant/Default,$(1))
  TITLE+= (Wi-Fi P2P support)
  VARIANT:=supplicant-p2p-internal
endef

define Package/wpa-supplicant-mesh/Default
$(call Package/wpa-supplicant/Default,$(1))
  DEPENDS+=@(!TARGET_uml||BROKEN)
  PROVIDES+=wpa-supplicant-mesh
endef

define Package/wpa-supplicant-mesh-openssl
$(call Package/wpa-supplicant-mesh/Default,$(1))
  TITLE+= (OpenSSL, 11s, SAE)
  VARIANT:=supplicant-mesh-openssl
  DEPENDS+=$(OPENSSL_DEPENDS)
endef

define Package/wpa-supplicant-mesh-wolfssl
$(call Package/wpa-supplicant-mesh/Default,$(1))
  TITLE+= (wolfSSL, 11s, SAE)
  VARIANT:=supplicant-mesh-wolfssl
  DEPENDS+=+PACKAGE_wpa-supplicant-mesh-wolfssl:libwolfssl
endef

define Package/wpa-supplicant-mesh-mbedtls
$(call Package/wpa-supplicant-mesh/Default,$(1))
  TITLE+= (mbedTLS, 11s, SAE)
  VARIANT:=supplicant-mesh-mbedtls
  DEPENDS+=+PACKAGE_wpa-supplicant-mesh-mbedtls:libmbedtls
endef

define Package/wpa-supplicant-basic
$(call Package/wpa-supplicant/Default,$(1))
  TITLE+= (11r, 11w)
  VARIANT:=supplicant-basic
endef

define Package/wpa-supplicant-mini
$(call Package/wpa-supplicant/Default,$(1))
  TITLE+= (minimal)
  VARIANT:=supplicant-mini
endef


define Package/hostapd-common
  TITLE:=hostapd/wpa_supplicant common support files
  SECTION:=net
  CATEGORY:=Network
  SUBMENU:=WirelessAPD
endef

define Package/hostapd-utils
  SECTION:=net
  CATEGORY:=Network
  SUBMENU:=WirelessAPD
  TITLE:=IEEE 802.1x Authenticator (utils)
  URL:=https://w1.fi/hostapd/
  DEPENDS:=@$(subst $(space),||,$(foreach pkg,$(HOSTAPD_PROVIDERS),PACKAGE_$(pkg)))
  VARIANT:=*
endef

define Package/hostapd-utils/description
 This package contains a command line utility to control the
 IEEE 802.1x/WPA/EAP/RADIUS Authenticator.
endef

define Package/wpa-cli
  SECTION:=net
  CATEGORY:=Network
  SUBMENU:=WirelessAPD
  DEPENDS:=@$(subst $(space),||,$(foreach pkg,$(SUPPLICANT_PROVIDERS),PACKAGE_$(pkg)))
  TITLE:=WPA Supplicant command line control utility
  VARIANT:=*
endef

define Package/eapol-test/Default
  TITLE:=802.1x auth test utility
  SECTION:=net
  SUBMENU:=WirelessAPD
  CATEGORY:=Network
  DEPENDS:=$(DRV_DEPENDS) $(CORE_DEPENDS)
endef

define Package/eapol-test
  $(call Package/eapol-test/Default,$(1))
  TITLE+= (built-in full)
  VARIANT:=supplicant-full-internal
endef

define Package/eapol-test-openssl
  $(call Package/eapol-test/Default,$(1))
  TITLE+= (OpenSSL full)
  VARIANT:=supplicant-full-openssl
  CONFLICTS:=$(filter-out eapol-test-openssl ,$(EAPOL_TEST_PROVIDERS))
  DEPENDS+=$(OPENSSL_DEPENDS)
  PROVIDES:=eapol-test
endef

define Package/eapol-test-wolfssl
  $(call Package/eapol-test/Default,$(1))
  TITLE+= (wolfSSL full)
  VARIANT:=supplicant-full-wolfssl
  CONFLICTS:=$(filter-out eapol-test-openssl ,$(filter-out eapol-test-wolfssl ,$(EAPOL_TEST_PROVIDERS)))
  DEPENDS+=+PACKAGE_eapol-test-wolfssl:libwolfssl
  PROVIDES:=eapol-test
endef

define Package/eapol-test-mbedtls
  $(call Package/eapol-test/Default,$(1))
  TITLE+= (mbedTLS full)
  VARIANT:=supplicant-full-mbedtls
  CONFLICTS:=$(filter-out eapol-test-openssl ,$(filter-out eapol-test-mbedtls ,$(EAPOL_TEST_PROVIDERS)))
  DEPENDS+=+PACKAGE_eapol-test-mbedtls:libmbedtls
  PROVIDES:=eapol-test
endef


ifneq ($(wildcard $(PKG_BUILD_DIR)/.config_*),$(subst .configured_,.config_,$(STAMP_CONFIGURED)))
  define Build/Configure/rebuild
	$(FIND) $(PKG_BUILD_DIR) -name \*.o -or -name \*.a | $(XARGS) rm -f
	rm -f $(PKG_BUILD_DIR)/hostapd/hostapd
	rm -f $(PKG_BUILD_DIR)/wpa_supplicant/wpa_supplicant
	rm -f $(PKG_BUILD_DIR)/.config_*
	touch $(subst .configured_,.config_,$(STAMP_CONFIGURED))
  endef
endif

define Build/Configure
	$(Build/Configure/rebuild)
	$(if $(wildcard ./files/hostapd-$(CONFIG_VARIANT).config), \
		$(CP) ./files/hostapd-$(CONFIG_VARIANT).config $(PKG_BUILD_DIR)/hostapd/.config \
	)
	$(if $(wildcard ./files/wpa_supplicant-$(CONFIG_VARIANT).config), \
		$(CP) ./files/wpa_supplicant-$(CONFIG_VARIANT).config $(PKG_BUILD_DIR)/wpa_supplicant/.config
	)
endef

TARGET_CPPFLAGS := \
	-I$(STAGING_DIR)/usr/include/libnl-tiny \
	-I$(PKG_BUILD_DIR)/src/crypto \
	$(TARGET_CPPFLAGS) \
	-DCONFIG_LIBNL20 \
	-D_GNU_SOURCE \
	$(if $(CONFIG_WPA_MSG_MIN_PRIORITY),-DCONFIG_MSG_MIN_PRIORITY=$(CONFIG_WPA_MSG_MIN_PRIORITY))

TARGET_LDFLAGS += -lubox -lubus -lblobmsg_json -lucode -lm -lnl-tiny -ludebug

ifdef CONFIG_WPA_ENABLE_WEP
    DRIVER_MAKEOPTS += CONFIG_WEP=y
endif

define Build/RunMake
	CFLAGS="$(TARGET_CPPFLAGS) $(TARGET_CFLAGS)" \
	$(MAKE) $(PKG_JOBS) -C $(PKG_BUILD_DIR)/$(1) \
		$(TARGET_CONFIGURE_OPTS) \
		$(DRIVER_MAKEOPTS) \
		LIBS="$(TARGET_LDFLAGS)" \
		LIBS_c="$(TARGET_LDFLAGS_C)" \
		AR="$(TARGET_CROSS)gcc-ar" \
		BCHECK= \
		$(if $(findstring s,$(OPENWRT_VERBOSE)),V=1) \
		$(2)
endef

define Build/Compile/wpad
	echo ` \
		$(call Build/RunMake,hostapd,-s MULTICALL=1 dump_cflags); \
		$(call Build/RunMake,wpa_supplicant,-s MULTICALL=1 dump_cflags) | \
		sed -e 's,-n ,,g' -e 's^$(TARGET_CFLAGS)^^' \
	` > $(PKG_BUILD_DIR)/.cflags
	sed -i 's/"/\\"/g' $(PKG_BUILD_DIR)/.cflags
	+$(call Build/RunMake,hostapd, \
		CFLAGS="$$$$(cat $(PKG_BUILD_DIR)/.cflags)" \
		MULTICALL=1 \
		hostapd_cli hostapd_multi.a \
	)
	+$(call Build/RunMake,wpa_supplicant, \
		CFLAGS="$$$$(cat $(PKG_BUILD_DIR)/.cflags)" \
		MULTICALL=1 \
		wpa_cli wpa_supplicant_multi.a \
	)
	+export MAKEFLAGS="$(MAKE_JOBSERVER)"; $(TARGET_CC) -o $(PKG_BUILD_DIR)/wpad \
		$(TARGET_CFLAGS) \
		./files/multicall.c \
		$(PKG_BUILD_DIR)/hostapd/hostapd_multi.a \
		$(PKG_BUILD_DIR)/wpa_supplicant/wpa_supplicant_multi.a \
		$(TARGET_LDFLAGS)
endef

define Build/Compile/hostapd
	+$(call Build/RunMake,hostapd, \
		hostapd hostapd_cli \
	)
endef

define Build/Compile/supplicant
	+$(call Build/RunMake,wpa_supplicant, \
		wpa_cli wpa_supplicant \
	)
endef

define Build/Compile/supplicant-full-internal
	+$(call Build/RunMake,wpa_supplicant, \
		eapol_test \
	)
endef

define Build/Compile/supplicant-full-openssl
	+$(call Build/RunMake,wpa_supplicant, \
		eapol_test \
	)
endef

define Build/Compile/supplicant-full-wolfssl
	+$(call Build/RunMake,wpa_supplicant, \
		eapol_test \
	)
endef

define Build/Compile/supplicant-full-mbedtls
	+$(call Build/RunMake,wpa_supplicant, \
		eapol_test \
	)
endef

define Build/Compile
	$(Build/Compile/$(LOCAL_TYPE))
	$(Build/Compile/$(BUILD_VARIANT))
endef

define Install/hostapd/full
	$(INSTALL_DIR) $(1)/etc/init.d $(1)/etc/config $(1)/etc/radius
	ln -sf hostapd $(1)/usr/sbin/hostapd-radius
	$(INSTALL_BIN) ./files/radius.init $(1)/etc/init.d/radius
	$(INSTALL_DATA) ./files/radius.config $(1)/etc/config/radius
	$(INSTALL_DATA) ./files/radius.clients $(1)/etc/radius/clients
	$(INSTALL_DATA) ./files/radius.users $(1)/etc/radius/users
endef

define Package/hostapd-full/conffiles
/etc/config/radius
/etc/radius
endef

ifeq ($(CONFIG_VARIANT),full)
Package/wpad-mesh-openssl/conffiles = $(Package/hostapd-full/conffiles)
Package/wpad-mesh-wolfssl/conffiles = $(Package/hostapd-full/conffiles)
Package/wpad-mesh-mbedtls/conffiles = $(Package/hostapd-full/conffiles)
Package/wpad/conffiles = $(Package/hostapd-full/conffiles)
Package/wpad-openssl/conffiles = $(Package/hostapd-full/conffiles)
Package/wpad-wolfssl/conffiles = $(Package/hostapd-full/conffiles)
Package/wpad-mbedtls/conffiles = $(Package/hostapd-full/conffiles)
Package/hostapd/conffiles = $(Package/hostapd-full/conffiles)
Package/hostapd-openssl/conffiles = $(Package/hostapd-full/conffiles)
Package/hostapd-wolfssl/conffiles = $(Package/hostapd-full/conffiles)
Package/hostapd-mbedtls/conffiles = $(Package/hostapd-full/conffiles)
endif

define Install/hostapd
	$(INSTALL_DIR) $(1)/usr/sbin $(1)/usr/share/hostap
	$(INSTALL_DATA) ./files/hostapd.uc $(1)/usr/share/hostap/
	$(if $(findstring full,$(CONFIG_VARIANT)),$(Install/hostapd/full))
endef

define Install/supplicant
	$(INSTALL_DIR) $(1)/usr/sbin $(1)/usr/share/hostap
	$(INSTALL_DATA) ./files/wpa_supplicant.uc $(1)/usr/share/hostap/
endef

define Package/hostapd-common/install
	$(INSTALL_DIR) \
	  $(1)/etc/capabilities \
	  $(1)/etc/rc.button \
	  $(1)/etc/hotplug.d/ieee80211 \
	  $(1)/etc/init.d $(1)/lib/netifd \
	  $(1)/usr/share/acl.d \
	  $(1)/usr/share/hostap
	$(INSTALL_BIN) ./files/dhcp-get-server.sh $(1)/lib/netifd/dhcp-get-server.sh
	$(INSTALL_BIN) ./files/wpad.init $(1)/etc/init.d/wpad
	$(INSTALL_BIN) ./files/wps-hotplug.sh $(1)/etc/rc.button/wps
	$(INSTALL_DATA) ./files/wpad_acl.json $(1)/usr/share/acl.d
	$(INSTALL_DATA) ./files/wpad.json $(1)/etc/capabilities
endef

define Package/hostapd/install
	$(call Install/hostapd,$(1))
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/hostapd/hostapd $(1)/usr/sbin/
endef
Package/hostapd-basic/install = $(Package/hostapd/install)
Package/hostapd-basic-openssl/install = $(Package/hostapd/install)
Package/hostapd-basic-wolfssl/install = $(Package/hostapd/install)
Package/hostapd-basic-mbedtls/install = $(Package/hostapd/install)
Package/hostapd-mini/install = $(Package/hostapd/install)
Package/hostapd-openssl/install = $(Package/hostapd/install)
Package/hostapd-wolfssl/install = $(Package/hostapd/install)
Package/hostapd-mbedtls/install = $(Package/hostapd/install)

ifneq ($(LOCAL_TYPE),supplicant)
  define Package/hostapd-utils/install
	$(INSTALL_DIR) $(1)/usr/sbin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/hostapd/hostapd_cli $(1)/usr/sbin/
  endef
endif

define Package/wpad/install
	$(call Install/hostapd,$(1))
	$(call Install/supplicant,$(1))
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/wpad $(1)/usr/sbin/
	$(LN) wpad $(1)/usr/sbin/hostapd
	$(LN) wpad $(1)/usr/sbin/wpa_supplicant
endef
Package/wpad-basic/install = $(Package/wpad/install)
Package/wpad-basic-openssl/install = $(Package/wpad/install)
Package/wpad-basic-wolfssl/install = $(Package/wpad/install)
Package/wpad-basic-mbedtls/install = $(Package/wpad/install)
Package/wpad-mini/install = $(Package/wpad/install)
Package/wpad-openssl/install = $(Package/wpad/install)
Package/wpad-wolfssl/install = $(Package/wpad/install)
Package/wpad-mbedtls/install = $(Package/wpad/install)
Package/wpad-mesh-openssl/install = $(Package/wpad/install)
Package/wpad-mesh-wolfssl/install = $(Package/wpad/install)
Package/wpad-mesh-mbedtls/install = $(Package/wpad/install)

define Package/wpa-supplicant/install
	$(call Install/supplicant,$(1))
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/wpa_supplicant/wpa_supplicant $(1)/usr/sbin/
endef
Package/wpa-supplicant-basic/install = $(Package/wpa-supplicant/install)
Package/wpa-supplicant-mini/install = $(Package/wpa-supplicant/install)
Package/wpa-supplicant-p2p/install = $(Package/wpa-supplicant/install)
Package/wpa-supplicant-openssl/install = $(Package/wpa-supplicant/install)
Package/wpa-supplicant-wolfssl/install = $(Package/wpa-supplicant/install)
Package/wpa-supplicant-mbedtls/install = $(Package/wpa-supplicant/install)
Package/wpa-supplicant-mesh-openssl/install = $(Package/wpa-supplicant/install)
Package/wpa-supplicant-mesh-wolfssl/install = $(Package/wpa-supplicant/install)
Package/wpa-supplicant-mesh-mbedtls/install = $(Package/wpa-supplicant/install)

ifneq ($(LOCAL_TYPE),hostapd)
  define Package/wpa-cli/install
	$(INSTALL_DIR) $(1)/usr/sbin
	$(CP) $(PKG_BUILD_DIR)/wpa_supplicant/wpa_cli $(1)/usr/sbin/
  endef
endif

ifeq ($(BUILD_VARIANT),supplicant-full-internal)
  define Package/eapol-test/install
	$(INSTALL_DIR) $(1)/usr/sbin
	$(CP) $(PKG_BUILD_DIR)/wpa_supplicant/eapol_test $(1)/usr/sbin/
  endef
endif

ifeq ($(BUILD_VARIANT),supplicant-full-openssl)
  define Package/eapol-test-openssl/install
	$(INSTALL_DIR) $(1)/usr/sbin
	$(CP) $(PKG_BUILD_DIR)/wpa_supplicant/eapol_test $(1)/usr/sbin/
  endef
endif

ifeq ($(BUILD_VARIANT),supplicant-full-wolfssl)
  define Package/eapol-test-wolfssl/install
	$(INSTALL_DIR) $(1)/usr/sbin
	$(CP) $(PKG_BUILD_DIR)/wpa_supplicant/eapol_test $(1)/usr/sbin/
  endef
endif

ifeq ($(BUILD_VARIANT),supplicant-full-mbedtls)
  define Package/eapol-test-mbedtls/install
	$(INSTALL_DIR) $(1)/usr/sbin
	$(CP) $(PKG_BUILD_DIR)/wpa_supplicant/eapol_test $(1)/usr/sbin/
  endef
endif

# Build hostapd-common before its dependents, to avoid
# spurious rebuilds when building multiple variants.
$(eval $(call BuildPackage,hostapd-common))
$(eval $(call BuildPackage,hostapd))
$(eval $(call BuildPackage,hostapd-basic))
$(eval $(call BuildPackage,hostapd-basic-openssl))
$(eval $(call BuildPackage,hostapd-basic-wolfssl))
$(eval $(call BuildPackage,hostapd-basic-mbedtls))
$(eval $(call BuildPackage,hostapd-mini))
$(eval $(call BuildPackage,hostapd-openssl))
$(eval $(call BuildPackage,hostapd-wolfssl))
$(eval $(call BuildPackage,hostapd-mbedtls))
$(eval $(call BuildPackage,wpad))
$(eval $(call BuildPackage,wpad-mesh-openssl))
$(eval $(call BuildPackage,wpad-mesh-wolfssl))
$(eval $(call BuildPackage,wpad-mesh-mbedtls))
$(eval $(call BuildPackage,wpad-basic))
$(eval $(call BuildPackage,wpad-basic-openssl))
$(eval $(call BuildPackage,wpad-basic-wolfssl))
$(eval $(call BuildPackage,wpad-basic-mbedtls))
$(eval $(call BuildPackage,wpad-mini))
$(eval $(call BuildPackage,wpad-openssl))
$(eval $(call BuildPackage,wpad-wolfssl))
$(eval $(call BuildPackage,wpad-mbedtls))
$(eval $(call BuildPackage,wpa-supplicant))
$(eval $(call BuildPackage,wpa-supplicant-mesh-openssl))
$(eval $(call BuildPackage,wpa-supplicant-mesh-wolfssl))
$(eval $(call BuildPackage,wpa-supplicant-mesh-mbedtls))
$(eval $(call BuildPackage,wpa-supplicant-basic))
$(eval $(call BuildPackage,wpa-supplicant-mini))
$(eval $(call BuildPackage,wpa-supplicant-p2p))
$(eval $(call BuildPackage,wpa-supplicant-openssl))
$(eval $(call BuildPackage,wpa-supplicant-wolfssl))
$(eval $(call BuildPackage,wpa-supplicant-mbedtls))
$(eval $(call BuildPackage,wpa-cli))
$(eval $(call BuildPackage,hostapd-utils))
$(eval $(call BuildPackage,eapol-test))
$(eval $(call BuildPackage,eapol-test-openssl))
$(eval $(call BuildPackage,eapol-test-wolfssl))
$(eval $(call BuildPackage,eapol-test-mbedtls))
