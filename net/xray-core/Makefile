include $(TOPDIR)/rules.mk

PKG_NAME:=xray-core
PKG_VERSION:=1.8.3
PKG_RELEASE:=2

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=https://codeload.github.com/XTLS/Xray-core/tar.gz/v$(PKG_VERSION)?
PKG_HASH:=bdfa65c15cd25f931745d9c70c753503db5d119ff11960ca7b3a2e19c4b0a8d1

PKG_MAINTAINER:=Tianling Shen <cnsztl@immortalwrt.org>
PKG_LICENSE:=MPL-2.0
PKG_LICENSE_FILES:=LICENSE

PKG_BUILD_DIR:=$(BUILD_DIR)/Xray-core-$(PKG_VERSION)
PKG_BUILD_PARALLEL:=1
PKG_USE_MIPS16:=0

include $(INCLUDE_DIR)/package.mk

# ---------------------------------------------------------------------------
# OpenWrt 19.07 ships golang 1.18, which cannot build xray 1.8.3 (needs Go >= 1.20).
# This package carries its own Go toolchain (downloaded once into dl/), so the
# golang package and other packages are left untouched.
#   - host arch is detected with uname -m (amd64 / arm64)
#   - target arch is mapped from OpenWrt ARCH (arm / mips / mipsel / mips64 ...)
# ---------------------------------------------------------------------------

GO_VER:=1.20.14
GO_HOST_ARCH:=$(shell uname -m)
GO_TARBALL_amd64:=go$(GO_VER).linux-amd64.tar.gz
GO_TARBALL_x86_64:=go$(GO_VER).linux-amd64.tar.gz
GO_TARBALL_aarch64:=go$(GO_VER).linux-arm64.tar.gz
GO_TARBALL_arm64:=go$(GO_VER).linux-arm64.tar.gz
GO_TARBALL:=$(GO_TARBALL_$(GO_HOST_ARCH))
GO_URL:=https://mirrors.aliyun.com/golang/$(GO_TARBALL)
GO_URL_FALLBACK:=https://go.dev/dl/$(GO_TARBALL)
GO_HOST_DIR:=$(BUILD_DIR)/go-$(GO_VER)-host

# OpenWrt ARCH -> Go GOARCH
GO_ARCH_arm:=arm
GO_ARCH_aarch64:=arm64
GO_ARCH_mips:=mips
GO_ARCH_mipsel:=mipsle
GO_ARCH_mips64:=mips64
GO_ARCH_mips64el:=mips64le
GO_ARCH_i386:=386
GO_ARCH_x86_64:=amd64
GO_ARCH_powerpc:=ppc64
GO_ARCH_powerpc64:=ppc64
GO_ARCH_riscv64:=riscv64
GOARCH_XRAY:=$(GO_ARCH_$(ARCH))

ifeq ($(GOARCH_XRAY),arm)
  GO_ARM_ENV:=GOARM=7
endif
ifeq ($(GOARCH_XRAY),mips)
  GO_MIPS_ENV:=GOMIPS=softfloat
endif
ifeq ($(GOARCH_XRAY),mipsle)
  GO_MIPS_ENV:=GOMIPS=softfloat
endif

GOPROXY_XRAY?=https://goproxy.cn,direct

define Build/Compile
	@set -e; \
	if [ ! -x "$(GO_HOST_DIR)/bin/go" ]; then \
		mkdir -p "$(GO_HOST_DIR)"; \
		if [ ! -f "$(DL_DIR)/$(GO_TARBALL)" ]; then \
			echo "Downloading $(GO_TARBALL) ..."; \
			curl -fsSL -o "$(DL_DIR)/$(GO_TARBALL).tmp" "$(GO_URL)" || \
			curl -fsSL -o "$(DL_DIR)/$(GO_TARBALL).tmp" "$(GO_URL_FALLBACK)"; \
			mv "$(DL_DIR)/$(GO_TARBALL).tmp" "$(DL_DIR)/$(GO_TARBALL)"; \
		fi; \
		tar -C "$(GO_HOST_DIR)" --strip-components=1 -xzf "$(DL_DIR)/$(GO_TARBALL)"; \
	fi; \
	cd "$(PKG_BUILD_DIR)" && \
	GOOS=linux GOARCH="$(GOARCH_XRAY)" $(GO_ARM_ENV) $(GO_MIPS_ENV) CGO_ENABLED=0 \
	GOPROXY="$(GOPROXY_XRAY)" \
	"$(GO_HOST_DIR)/bin/go" build -trimpath -buildvcs=false \
		-ldflags "-s -w -X github.com/xtls/xray-core/core.build=OpenWrt -X github.com/xtls/xray-core/core.version=$(PKG_VERSION)" \
		-o "$(PKG_BUILD_DIR)/xray" ./main
endef

define Package/xray/template
  TITLE:=A platform for building proxies to bypass network restrictions
  SECTION:=net
  CATEGORY:=Network
  URL:=https://xtls.github.io
endef

define Package/xray-core
  $(call Package/xray/template)
  DEPENDS:=+ca-bundle
endef

define Package/xray-example
  $(call Package/xray/template)
  TITLE+= (example configs)
  DEPENDS:=xray-core
  PKGARCH:=all
endef

define Package/xray/description
  Xray, Penetrates Everything. It helps you to build your own computer network.
  It secures your network connections and thus protects your privacy.
endef

define Package/xray-core/description
  $(call Package/xray/description)
endef

define Package/xray-example/description
  $(call Package/xray/description)

  This includes example configuration files for xray-core.
endef

define Package/xray-core/conffiles
/etc/xray/
/etc/config/xray
endef

define Package/xray-core/install
	$(INSTALL_DIR) $(1)/usr/bin/
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/xray $(1)/usr/bin/xray

	$(INSTALL_DIR) $(1)/etc/xray/
	$(INSTALL_DATA) $(CURDIR)/files/config.json.example $(1)/etc/xray/

	$(INSTALL_DIR) $(1)/etc/config/
	$(INSTALL_CONF) $(CURDIR)/files/xray.conf $(1)/etc/config/xray
	$(INSTALL_DIR) $(1)/etc/init.d/
	$(INSTALL_BIN) $(CURDIR)/files/xray.init $(1)/etc/init.d/xray
endef

define Package/xray-example/install
	$(INSTALL_DIR) $(1)/etc/xray/
	$(INSTALL_DATA) $(CURDIR)/files/vpoint_socks_vmess.json $(1)/etc/xray/
	$(INSTALL_DATA) $(CURDIR)/files/vpoint_vmess_freedom.json $(1)/etc/xray/
endef

$(eval $(call BuildPackage,xray-core))
$(eval $(call BuildPackage,xray-example))
