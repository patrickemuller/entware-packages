# Based partially on the versions of el1n and Federico Di Marco
# Entware specific: host compilation disabled, run hamcorebuilder on target instead.

include $(TOPDIR)/rules.mk

PKG_NAME:=softethervpn5
PKG_VERSION:=5.01.9672
PKG_RELEASE:=1

PKG_MAINTAINER:=Andy Walsh <andy.walsh44+github@gmail.com>
PKG_LICENSE:=GPL-2.0
PKG_LICENSE_FILES:=COPYING

PKG_SOURCE_URL:=https://github.com/SoftEtherVPN/SoftEtherVPN/releases/download/$(PKG_VERSION)/
PKG_SOURCE:=softether-vpn-src-$(PKG_VERSION).tar.gz
PKG_HASH:=7c437d4d02d7e2a936b4c1ff7bc8f5abbf16786746deffa92d5f5f2fd7ba04fb

#HOST_BUILD_DIR:=$(BUILD_DIR_HOST)/SoftEtherVPN-$(PKG_VERSION)
PKG_BUILD_DIR:=$(BUILD_DIR)/SoftEtherVPN-$(PKG_VERSION)

#HOST_BUILD_DEPENDS:=ncurses/host readline/host
#PKG_BUILD_DEPENDS:=softethervpn5/host

include $(INCLUDE_DIR)/package.mk
#include $(INCLUDE_DIR)/host-build.mk
include $(INCLUDE_DIR)/nls.mk
include $(INCLUDE_DIR)/cmake.mk

define Package/softethervpn5/Default
  SECTION:=net
  CATEGORY:=Network
  SUBMENU:=VPN
  TITLE:=softethervpn5 v$(PKG_VERSION)
  URL:=http://www.softether.org/
endef

define Package/softethervpn5/Default/description
  SoftEther VPN supports SSL-VPN, OpenVPN, L2TP, EtherIP, L2TPv3 and IPsec as a single VPN software.
  SoftEther VPN is not only an alternative VPN server to existing VPN products (OpenVPN, IPsec and MS-SSTP),
  but has also original strong SSL-VPN protocol to penetrate any kinds of firewalls.
  Guide: https://wordpress.tirlins.com/2015/03/setting-up-softether-vpn-on-openwrt/
endef

define Package/softethervpn5-libs
  $(call Package/softethervpn5/Default)
  DEPENDS:=+libpthread +librt +libreadline +libopenssl +libncurses +zlib $(ICONV_DEPENDS) +USE_GLIBC:gconv-modules
  TITLE+= libs
  HIDDEN:=1
endef

define Package/softethervpn5-server
  $(call Package/softethervpn5/Default)
  TITLE+= server
  DEPENDS:= +softethervpn5-libs
endef
define Package/softethervpn5-server/description
  $(call Package/softethervpn5/Default/description)

  Provides the vpnserver (daemon).
endef

define Package/softethervpn5-bridge
  $(call Package/softethervpn5/Default)
  TITLE+= bridge
  DEPENDS:= +softethervpn5-libs  
endef
define Package/softethervpn5-bridge/description
  $(call Package/softethervpn5/Default/description)

  Provides the vpnbridge (daemon).
endef

define Package/softethervpn5-client
  $(call Package/softethervpn5/Default)
  TITLE+= client
  DEPENDS:= +softethervpn5-libs  
endef
define Package/softethervpn5-client/description
  $(call Package/softethervpn5/Default/description)

  Provides the vpnclient.
endef

#export USE_MUSL=YES
# BUG: outdated host/include/elf.h
HOST_CFLAGS += $(FPIC) -DAT_HWCAP2=26
TARGET_CFLAGS += $(FPIC) -I$(STAGING_DIR)/opt/include
CMAKE_OPTIONS = -DICONV_LIB_PATH="$(ICONV_PREFIX)/lib"

# static build for host (hamcorebuilder), avoid -fpic on ncurses/host and shared libs can't be found on host
define Host/Prepare
	$(Host/Prepare/Default)
	$(SED) 's,SHARED,STATIC,g' $(HOST_BUILD_DIR)/src/Mayaqua/CMakeLists.txt
	$(SED) 's,SHARED,STATIC,g' $(HOST_BUILD_DIR)/src/Cedar/CMakeLists.txt
	$(SED) 's,readline,libreadline.a,g' $(HOST_BUILD_DIR)/src/Cedar/CMakeLists.txt
endef

define Host/Compile
	$(call Host/Compile/Default,hamcorebuilder)
endef

define Host/Install
	$(INSTALL_DIR) $(STAGING_DIR_HOSTPKG)/bin/
	$(INSTALL_BIN) $(HOST_BUILD_DIR)/tmp/hamcorebuilder $(STAGING_DIR_HOSTPKG)/bin/
endef

define Build/Compile
	$(call Build/Compile/Default,vpnserver vpnbridge vpnclient vpncmd hamcorebuilder)
endef

define Build/Install
endef

define Package/softethervpn5-libs/install
	$(INSTALL_DIR) $(1)/opt/lib
	$(CP) $(PKG_BUILD_DIR)/build/libcedar.so $(1)/opt/lib/
	$(CP) $(PKG_BUILD_DIR)/build/libmayaqua.so $(1)/opt/lib/
	$(INSTALL_DIR) $(1)/opt/libexec/softethervpn
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/build/vpncmd $(1)/opt/libexec/softethervpn/
	$(INSTALL_BIN) files/launcher.sh $(1)/opt/libexec/softethervpn/
	$(INSTALL_DATA) files/dummy $(1)/opt/libexec/softethervpn/lang.config
	$(CP) $(PKG_BUILD_DIR)/src/bin/hamcore $(1)/opt/libexec/softethervpn/
	$(INSTALL_DIR) $(1)/opt/bin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/tmp/hamcorebuilder $(1)/opt/bin
	$(LN) ../../opt/libexec/softethervpn/launcher.sh $(1)/opt/bin/vpncmd
endef

define Package/softethervpn5-server/install
	$(INSTALL_DIR) $(1)/opt/libexec/softethervpn
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/build/vpnserver $(1)/opt/libexec/softethervpn/
	$(INSTALL_DATA) files/dummy $(1)/opt/libexec/softethervpn/vpn_server.config
	$(INSTALL_DIR) $(1)/opt/etc/init.d
	$(INSTALL_BIN) files/S05vpnserver $(1)/opt/etc/init.d
endef

define Package/softethervpn5-bridge/install
	$(INSTALL_DIR) $(1)/opt/libexec/softethervpn
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/build/vpnbridge $(1)/opt/libexec/softethervpn/
	$(INSTALL_DATA) files/dummy $(1)/opt/libexec/softethervpn/vpn_bridge.config
	$(INSTALL_DIR) $(1)/opt/etc/init.d
	$(INSTALL_BIN) files/S05vpnbridge $(1)/opt/etc/init.d
endef

define Package/softethervpn5-client/install
	$(INSTALL_DIR) $(1)/opt/libexec/softethervpn
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/build/vpnclient $(1)/opt/libexec/softethervpn/
	$(INSTALL_DATA) files/dummy $(1)/opt/libexec/softethervpn/vpn_client.config
	$(INSTALL_DIR) $(1)/opt/etc/init.d
	$(INSTALL_BIN) files/S05vpnclient $(1)/opt/etc/init.d
endef

define Package/softethervpn5-libs/postinst
#!/bin/sh

echo 'Creating hamcore.se2 archive... '
GCONV_PATH=/opt/lib/gconv hamcorebuilder /opt/libexec/softethervpn/hamcore \
/opt/libexec/softethervpn/hamcore.se2 && rm -fr /opt/libexec/softethervpn/hamcore
endef

define Package/softethervpn5-libs/prerm
#!/bin/sh

echo 'Deleting hamcore.se2 archive... '
rm -fr /opt/libexec/softethervpn/hamcore.se2
endef

define Package/softethervpn5-server/conffiles
/opt/libexec/softethervpn/vpn_server.config
/opt/libexec/softethervpn/lang.config
endef

define Package/softethervpn5-bridge/conffiles
/opt/libexec/softethervpn/vpn_bridge.config
/opt/libexec/softethervpn/lang.config
endef

define Package/softethervpn5-client/conffiles
/opt/libexec/softethervpn/vpn_client.config
/opt/libexec/softethervpn/lang.config
endef

#$(eval $(call HostBuild))
$(eval $(call BuildPackage,softethervpn5-libs))
$(eval $(call BuildPackage,softethervpn5-server))
$(eval $(call BuildPackage,softethervpn5-bridge))
$(eval $(call BuildPackage,softethervpn5-client))
