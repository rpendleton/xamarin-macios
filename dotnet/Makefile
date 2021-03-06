TOP = ..

include $(TOP)/Make.config
include $(TOP)/mk/rules.mk

DOTNET_MANIFESTS_PATH=$(DOTNET6_DIR)/sdk-manifests/$(DOTNET6_VERSION_BAND)
DOTNET_PACKS_PATH=$(DOTNET6_DIR)/packs
TMP_PKG_DIR=_pkg

DOTNET_PLATFORMS_UPPERCASE:=$(shell echo $(DOTNET_PLATFORMS) | tr a-z A-Z)

# Create variables prefixed with the correctly cased platform name from the upper-cased platform name. This makes some of the next sections somewhat simpler.
$(foreach platform,$(DOTNET_PLATFORMS),$(eval $(platform)_NUGET_VERSION_NO_METADATA:=$($(shell echo $(platform) | tr a-z A-Z)_NUGET_VERSION_NO_METADATA)))

define DefineTargets
$(1)_NUGET_TARGETS = \
	$(DOTNET_DESTDIR)/Microsoft.$(1).Sdk/Sdk/AutoImport.props \
	$(DOTNET_DESTDIR)/Microsoft.$(1).Sdk/Sdk/Sdk.props \
	$(DOTNET_DESTDIR)/Microsoft.$(1).Sdk/targets/Microsoft.$(1).Sdk.SupportedTargetPlatforms.props \
	$(DOTNET_DESTDIR)/Microsoft.$(1).Sdk/targets/Microsoft.$(1).Sdk.DefaultItems.props \
	$(DOTNET_DESTDIR)/Microsoft.$(1).Sdk/targets/Microsoft.$(1).Sdk.props \
	$(DOTNET_DESTDIR)/Microsoft.$(1).Sdk/targets/Microsoft.$(1).Sdk.Versions.props \
	$(DOTNET_DESTDIR)/Microsoft.$(1).Sdk/targets/Microsoft.$(1).Sdk.targets \
	$(DOTNET_DESTDIR)/Microsoft.$(1).Sdk/targets/Xamarin.Shared.Sdk.DefaultItems.targets \
	$(DOTNET_DESTDIR)/Microsoft.$(1).Sdk/targets/Xamarin.Shared.Sdk.TargetFrameworkInference.props \
	$(DOTNET_DESTDIR)/Microsoft.$(1).Sdk/targets/Xamarin.Shared.Sdk.props \
	$(DOTNET_DESTDIR)/Microsoft.$(1).Sdk/targets/Xamarin.Shared.Sdk.targets \

endef
$(foreach platform,$(DOTNET_PLATFORMS),$(eval $(call DefineTargets,$(platform))))

DIRECTORIES += \
	$(DOTNET_NUPKG_DIR) \
	$(DOTNET_PKG_DIR) \
	$(DOTNET_FEED_DIR) \
	$(foreach platform,$(DOTNET_PLATFORMS),$(DOTNET_DESTDIR)/Microsoft.$(platform).Sdk/Sdk) \
	$(foreach platform,$(DOTNET_PLATFORMS),$(DOTNET_DESTDIR)/Microsoft.$(platform).Sdk/targets) \
	$(DOTNET_MANIFESTS_PATH) \
	$(DOTNET_PACKS_PATH) \
	$(foreach platform,$(DOTNET_PLATFORMS),$(DOTNET_PACKS_PATH)/Microsoft.$(platform).Sdk) \
	$(foreach platform,$(DOTNET_PLATFORMS),$(DOTNET_PACKS_PATH)/Microsoft.$(platform).Ref) \
	$(TMP_PKG_DIR) \

$(DIRECTORIES):
	$(Q) mkdir -p $@

CURRENT_HASH_LONG:=$(shell git log -1 --pretty=%H)

$(DOTNET_DESTDIR)/Microsoft.%: Microsoft.% | $(foreach platform,$(DOTNET_PLATFORMS),$(DOTNET_DESTDIR)/Microsoft.$(platform).Sdk/Sdk $(DOTNET_DESTDIR)/Microsoft.$(platform).Sdk/targets)
	$(Q) $(CP) $< $@

define CopyTargets
$$(DOTNET_DESTDIR)/Microsoft.$(1).Sdk/targets/%: targets/% | $$(DOTNET_DESTDIR)/Microsoft.$(1).Sdk/targets
	$$(Q) $$(CP) $$< $$@
endef
$(foreach platform,$(DOTNET_PLATFORMS),$(eval $(call CopyTargets,$(platform))))

define VersionsTemplate
targets/Microsoft.$(1).Sdk.Versions.props: targets/Microsoft.Sdk.Versions.template.props Makefile $(TOP)/Make.config.inc $(TOP)/.git/HEAD $(TOP)/.git/index
	$$(Q_GEN) sed \
		-e "s/@PLATFORM@/$(1)/g" \
		-e "s/@NUGET_VERSION_NO_METADATA@/$$($(2)_NUGET_VERSION_NO_METADATA)/g" \
		-e "s/@NUGET_VERSION_FULL@/$$($(2)_NUGET_VERSION_FULL)/g" \
		-e "s/@DEFAULT_TARGET_PLATFORM_VERSION@/$$(DEFAULT_TARGET_PLATFORM_VERSION_$(2))/g" \
		-e "s/@CURRENT_BRANCH@/$$(CURRENT_BRANCH_SED_ESCAPED)/g" \
		-e "s/@CURRENT_HASH_LONG@/$$(CURRENT_HASH_LONG)/g" \
		$$< > $$@.tmp
	$$(Q) mv $$@.tmp $$@

Microsoft.$1.Sdk/targets/Microsoft.$1.Sdk.Versions.props: targets/Microsoft.$1.Sdk.Versions.props
	$$(Q) $$(CP) $$< $$@
endef
$(foreach platform,$(DOTNET_PLATFORMS),$(eval $(call VersionsTemplate,$(platform),$(shell echo $(platform) | tr a-z A-Z))))

define DefaultItems
Microsoft.$1.Sdk/targets/Microsoft.$1.Sdk.DefaultItems.props: targets/Microsoft.Sdk.DefaultItems.template.props Makefile
	$$(Q_GEN) sed \
		-e "s/@PLATFORM@/$1/g" \
		$$< > $$@
endef
$(foreach platform,$(DOTNET_PLATFORMS),$(eval $(call DefaultItems,$(platform))))

define SupportedTargetPlatforms
Microsoft.$(1).Sdk/targets/Microsoft.$(1).Sdk.SupportedTargetPlatforms.props: $(TOP)/Versions-ios.plist.in $(TOP)/Versions-mac.plist.in Makefile ./generate-target-platforms.csharp Makefile
	$(Q) rm -f $$@.tmp
	$(Q) ./generate-target-platforms.csharp $(1) $$@.tmp
	$(Q) mv $$@.tmp $$@
endef
$(foreach platform,$(DOTNET_PLATFORMS),$(eval $(call SupportedTargetPlatforms,$(platform))))

define WorkloadTargets
Microsoft.NET.Workload.$(1)/WorkloadManifest.json: targets/WorkloadManifest.template.json Makefile $(TOP)/Make.config.inc $(TOP)/.git/HEAD $(TOP)/.git/index
	$$(Q) rm -f $$@.tmp
	$$(Q_GEN) sed \
		-e "s/@VERSION@/$3/g" \
		-e "s/@PLATFORM_LOWERCASE@/$2/g" \
		-e "s/@PLATFORM@/$1/g" \
		$$< > $$@.tmp
	$$(Q) mv $$@.tmp $$@

Microsoft.NET.Workload.$(1)/LICENSE: $(TOP)/LICENSE
	$$(Q) $(CP) $$< $$@

WORKLOAD_TARGETS += Microsoft.NET.Workload.$(1)/WorkloadManifest.json
WORKLOAD_TARGETS += Microsoft.NET.Workload.$(1)/LICENSE
endef
$(foreach platform,$(DOTNET_PLATFORMS),$(eval $(call WorkloadTargets,$(platform),$(shell echo $(platform) | tr A-Z a-z),$($(platform)_NUGET_VERSION_NO_METADATA))))

TEMPLATED_FILES = \
	$(foreach platform,$(DOTNET_PLATFORMS),Microsoft.$(platform).Sdk/targets/Microsoft.$(platform).Sdk.Versions.props) \
	$(foreach platform,$(DOTNET_PLATFORMS),Microsoft.$(platform).Sdk/targets/Microsoft.$(platform).Sdk.SupportedTargetPlatforms.props) \

nupkgs/$(IOS_WINDOWS_NUGET).%.nupkg: CURRENT_VERSION_NO_METADATA=$(IOS_WINDOWS_NUGET_VERSION_NO_METADATA)
nupkgs/$(IOS_WINDOWS_NUGET).%.nupkg: CURRENT_VERSION_FULL=$(IOS_WINDOWS_NUGET_VERSION_FULL)

define NupkgDefinition
nupkgs/$($(1)_NUGET).%.nupkg: CURRENT_VERSION_NO_METADATA=$($(1)_NUGET_VERSION_NO_METADATA)
nupkgs/$($(1)_NUGET).%.nupkg: CURRENT_VERSION_FULL=$($(1)_NUGET_VERSION_FULL)
endef
$(foreach platform,$(DOTNET_PLATFORMS),$(eval $(call NupkgDefinition,$(platform))))

# Create the nuget in a temporary directory (nupkgs/)
define CreateNuGetTemplate
nupkgs/$(1).$(2)+$(NUGET_BUILD_METADATA).nupkg: $(TEMPLATED_FILES) $(3) package/$(1)/package.csproj $(wildcard package/*.csproj) $(wildcard $(DOTNET_DESTDIR)/$(1)/* $(DOTNET_DESTDIR)/$(1)/*/* $(DOTNET_DESTDIR)/$(1)/*/*/* $(DOTNET_DESTDIR)/$(1)/*/*/*/*) global.json
	@# Delete any versions of the nuget we're building
	$$(Q) rm -f nupkgs/$(1).*.nupkg
	$$(Q_PACK) $(DOTNET6) pack package/$(1)/package.csproj --output "$$(dir $$@)" $(DOTNET_PACK_VERBOSITY)
	@# Nuget pack doesn't add the metadata to the filename, but we want that, so rename nuget to contain the full name
	$$(Q) mv "nupkgs/$(1).$(2).nupkg" "$$@"
	@# Clean the local feed
	$$(Q_NUGET_DEL) if test -d $(DOTNET_FEED_DIR)/$(shell echo $(1) | tr A-Z a-z)/$(2); then nuget delete $(1) $(2) -source $(abspath $(DOTNET_FEED_DIR)) -NonInteractive $(NUGET_VERBOSITY); fi
	@# Add the nupkg to our local feed
	$$(Q_NUGET_ADD) nuget add "$$@" -source $(DOTNET_FEED_DIR) -NonInteractive $(NUGET_VERBOSITY)
endef

define CreateWindowsNuGetTemplate
nupkgs/$(1).$(2)+$(NUGET_BUILD_METADATA).nupkg: $(3) package/$(1)/package.csproj $(wildcard package/*.csproj) $(wildcard $(DOTNET_DESTDIR)/$(1)/* $(DOTNET_DESTDIR)/$(1)/*/* $(DOTNET_DESTDIR)/$(1)/*/*/* $(DOTNET_DESTDIR)/$(1)/*/*/*/*) global.json
	@# Delete any versions of the nuget we're building
	$$(Q) rm -f nupkgs/$(1).*.nupkg
	$$(Q_PACK) $(DOTNET6) pack package/$(1)/package.csproj --output "$$(dir $$@)" $(DOTNET_PACK_VERBOSITY)
	@# Nuget pack doesn't add the metadata to the filename, but we want that, so rename nuget to contain the full name
	$$(Q) mv "nupkgs/$(1).$(2).nupkg" "$$@"
	@# Clean the local feed
	$$(Q_NUGET_DEL) if test -d $(DOTNET_FEED_DIR)/$(shell echo $(1) | tr A-Z a-z)/$(2); then nuget delete $(1) $(2) -source $(abspath $(DOTNET_FEED_DIR)) -NonInteractive $(NUGET_VERBOSITY); fi
	@# Add the nupkg to our local feed
	$$(Q_NUGET_ADD) nuget add "$$@" -source $(DOTNET_FEED_DIR) -NonInteractive $(NUGET_VERBOSITY)
endef

# Create the NuGet packaging targets. It's amazing what make allows you to do...
$(foreach platform,$(DOTNET_PLATFORMS),$(eval $(call CreateNuGetTemplate,Microsoft.$(platform).Sdk,$($(platform)_NUGET_VERSION_NO_METADATA),$($(platform)_NUGET_TARGETS))))
$(eval $(call CreateWindowsNuGetTemplate,Microsoft.iOS.Windows.Sdk,$(IOS_WINDOWS_NUGET_VERSION_NO_METADATA),$(IOS_WINDOWS_NUGET_TARGETS)))
$(foreach platform,$(DOTNET_PLATFORMS),$(eval $(call CreateNuGetTemplate,Microsoft.$(platform).Ref,$($(platform)_NUGET_VERSION_NO_METADATA))))
$(foreach platform,$(DOTNET_PLATFORMS),$(foreach rid,$(DOTNET_$(platform)_RUNTIME_IDENTIFIERS),$(eval $(call CreateNuGetTemplate,Microsoft.$(platform).Runtime.$(rid),$($(platform)_NUGET_VERSION_NO_METADATA)))))

# Copy the nuget from the temporary directory into the final directory
$(DOTNET_NUPKG_DIR)/%.nupkg: nupkgs/%.nupkg | $(DOTNET_NUPKG_DIR)
	$(Q) $(CP) $< $@

SDK_PACK_IOS_WINDOWS = $(DOTNET_NUPKG_DIR)/$(IOS_WINDOWS_NUGET).Sdk.$(IOS_WINDOWS_NUGET_VERSION_FULL).nupkg
ifdef ENABLE_DOTNET_WINDOWS
SDK_PACKS += $(SDK_PACK_IOS_WINDOWS)
endif

pack-ios-windows: $(SDK_PACK_IOS_WINDOWS)

define PacksDefinitions
RUNTIME_PACKS_$(1) = $$(foreach rid,$$(DOTNET_$(1)_RUNTIME_IDENTIFIERS),$(DOTNET_NUPKG_DIR)/$($(1)_NUGET).Runtime.$$(rid).$($(1)_NUGET_VERSION_FULL).nupkg)
RUNTIME_PACKS += $$(RUNTIME_PACKS_$(1))
REF_PACKS_$(1) = $(DOTNET_NUPKG_DIR)/$($(1)_NUGET).Ref.$($(1)_NUGET_VERSION_FULL).nupkg
REF_PACKS += $$(REF_PACKS_$(1))
SDK_PACKS_$(1) = $(DOTNET_NUPKG_DIR)/$($(1)_NUGET).Sdk.$($(1)_NUGET_VERSION_FULL).nupkg
SDK_PACKS += $$(SDK_PACKS_$(1))
pack-$(shell echo $(1) | tr A-Z a-z): $$(RUNTIME_PACKS_$(1)) $$(REF_PACKS_$(1)) $$(SDK_PACKS_$(1))
endef
$(foreach platform,$(DOTNET_PLATFORMS_UPPERCASE),$(eval $(call PacksDefinitions,$(platform))))

TARGETS += $(RUNTIME_PACKS) $(REF_PACKS) $(SDK_PACKS)

define InstallWorkload
$(DOTNET_MANIFESTS_PATH)/Microsoft.NET.Workload.$1: | $(DOTNET_MANIFESTS_PATH)
	$$(Q_LN) ln -Fhs $$(abspath Microsoft.NET.Workload.$1) $$(abspath $$@)

$(DOTNET_PACKS_PATH)/Microsoft.$1.Sdk/$2: | $(DOTNET_PACKS_PATH)/Microsoft.$1.Sdk
	$$(Q_LN) ln -Fhs $$(abspath $(DOTNET_DESTDIR)/Microsoft.$1.Sdk) $$(abspath $$@)

$(DOTNET_PACKS_PATH)/Microsoft.$1.Ref/$2: | $(DOTNET_PACKS_PATH)/Microsoft.$1.Ref
	$$(Q_LN) ln -Fhs $$(abspath $(DOTNET_DESTDIR)/Microsoft.$1.Ref) $$(abspath $$@)

WORKLOAD_TARGETS += \
	$(DOTNET_MANIFESTS_PATH)/Microsoft.NET.Workload.$1 \
	$(DOTNET_PACKS_PATH)/Microsoft.$1.Sdk/$2 \
	$(DOTNET_PACKS_PATH)/Microsoft.$1.Ref/$2
endef
$(foreach platform,$(DOTNET_PLATFORMS),$(eval $(call InstallWorkload,$(platform),$($(platform)_NUGET_VERSION_NO_METADATA))))

# We create three packages: one for the workload, one for the sdk package, and one for the ref package,
# and then bundle them into a single pkg for distribution (named Microsoft.<platform>.Bundle.<version>.pkg).
# This way we can create and publish separate updates for each workload/sdk/ref package later if we want to.
define CreatePackage
# The workload package
$(TMP_PKG_DIR)/Microsoft.$1.Workload.$2.pkg: $($(1)_NUGET_TARGETS) $(WORKLOAD_TARGETS) Makefile | $(TMP_PKG_DIR)
	$$(Q) rm -f $$@
	$$(Q) rm -rf tmpdir/Microsoft.$1.Workload.$2/
	$$(Q) mkdir -p tmpdir/Microsoft.$1.Workload.$2/usr/local/share/dotnet/sdk-manifests/$(DOTNET6_VERSION_BAND)/
	$$(Q) $$(CP) -r Microsoft.NET.Workload.$1 tmpdir/Microsoft.$1.Workload.$2/usr/local/share/dotnet/sdk-manifests/$(DOTNET6_VERSION_BAND)/
	$$(Q_GEN) pkgbuild --quiet --version '$2' --root tmpdir/Microsoft.$1.Workload.$2 --component-plist PackageInfo.plist  --install-location / --identifier com.microsoft.net.$3.workload.pkg $$@.tmp
	$$(Q) mv $$@.tmp $$@

# The sdk package
$(TMP_PKG_DIR)/Microsoft.$1.Sdk.$2.pkg: $(REF_PACK_$(4)) | $(TMP_PKG_DIR)
	$$(Q) rm -f $$@
	$$(Q) rm -rf tmpdir/Microsoft.$1.Sdk.$2/
	$$(Q) mkdir -p tmpdir/Microsoft.$1.Sdk.$2/usr/local/share/dotnet/packs/Microsoft.$1.Sdk/$2/
	$$(Q) $$(CP) -r $(DOTNET_DESTDIR)/Microsoft.$1.Sdk/ tmpdir/Microsoft.$1.Sdk.$2/usr/local/share/dotnet/packs/Microsoft.$1.Sdk/$2/
	$$(Q_GEN) pkgbuild --quiet --version '$2' --root tmpdir/Microsoft.$1.Sdk.$2 --component-plist PackageInfo.plist --install-location / --identifier com.microsoft.net.$3.sdk.pkg $$@.tmp
	$$(Q) mv $$@.tmp $$@

# The ref package
$(TMP_PKG_DIR)/Microsoft.$1.Ref.$2.pkg: $(SDK_PACK_$(4)) | $(TMP_PKG_DIR)
	$$(Q) rm -f $$@
	$$(Q) rm -rf tmpdir/Microsoft.$1.Ref.$2/
	$$(Q) mkdir -p tmpdir/Microsoft.$1.Ref.$2/usr/local/share/dotnet/packs/Microsoft.$1.Ref/$2/
	$$(Q) $$(CP) -r $(DOTNET_DESTDIR)/Microsoft.$1.Ref/ tmpdir/Microsoft.$1.Ref.$2/usr/local/share/dotnet/packs/Microsoft.$1.Ref/$2/
	$$(Q_GEN) pkgbuild --quiet --version '$2' --root tmpdir/Microsoft.$1.Ref.$2 --component-plist PackageInfo.plist --install-location / --identifier com.microsoft.net.$3.ref.pkg $$@.tmp
	$$(Q) mv $$@.tmp $$@

# The final bundle package for distribution
$(TMP_PKG_DIR)/Microsoft.$1.Bundle.$2.pkg: $(TMP_PKG_DIR)/Microsoft.$1.Workload.$2.pkg $(TMP_PKG_DIR)/Microsoft.$1.Sdk.$2.pkg $(TMP_PKG_DIR)/Microsoft.$1.Ref.$2.pkg
	$$(Q) rm -f $$@
	$$(Q_GEN) productbuild \
		--quiet \
		--identifier com.microsoft.net.$3.pkg \
		--version '$2' \
		--package $(TMP_PKG_DIR)/Microsoft.$1.Workload.$2.pkg \
		--package $(TMP_PKG_DIR)/Microsoft.$1.Sdk.$2.pkg \
		--package $(TMP_PKG_DIR)/Microsoft.$1.Ref.$2.pkg \
		$$@.tmp
	$$(Q) mv $$@.tmp $$@

# Copy the bundle package from our temporary directory to the target directory
$(DOTNET_PKG_DIR)/%: $(TMP_PKG_DIR)/% | $(DOTNET_PKG_DIR)
	$$(Q) $(CP) $$< $$@

PACKAGE_TARGETS += $(DOTNET_PKG_DIR)/Microsoft.$1.Bundle.$2.pkg

$(TMP_PKG_DIR)/Microsoft.$1.Bundle.$2.zip: $($(1)_NUGET_TARGETS) $(WORKLOAD_TARGETS) Makefile $(REF_PACK_$(4)) $(SDK_PACK_$(4)) | $(TMP_PKG_DIR)
	$$(Q) rm -rf $$@ $$@.tmpdir $$@.tmp
	$$(Q) mkdir -p $$@.tmpdir/dotnet/sdk-manifests/$(DOTNET6_VERSION_BAND)/
	$$(Q) mkdir -p $$@.tmpdir/dotnet/packs/Microsoft.$1.Sdk
	$$(Q) mkdir -p $$@.tmpdir/dotnet/packs/Microsoft.$1.Ref
	$$(Q) $(CP) -r Microsoft.NET.Workload.$1 $$@.tmpdir/dotnet/sdk-manifests/$(DOTNET6_VERSION_BAND)/
	$$(Q) $(CP) -r $(DOTNET_DESTDIR)/Microsoft.$1.Sdk $$@.tmpdir/dotnet/packs/Microsoft.$1.Sdk/$2
	$$(Q) $(CP) -r $(DOTNET_DESTDIR)/Microsoft.$1.Ref $$@.tmpdir/dotnet/packs/Microsoft.$1.Ref/$2
	$$(Q_GEN) cd $$@.tmpdir && zip -9rq $$(abspath $$@.tmp) .
	$$(Q) mv $$@.tmp $$@
	$$(Q) echo Created $$@

PACKAGE_TARGETS += $(DOTNET_PKG_DIR)/Microsoft.$1.Bundle.$2.zip
endef
$(foreach platform,$(DOTNET_PLATFORMS),$(eval $(call CreatePackage,$(platform),$($(platform)_NUGET_VERSION_NO_METADATA),$(shell echo $(platform) | tr A-Z a-z),$(shell echo $(platform) | tr a-z A-Z))))

define CreateWindowsBundle
$(TMP_PKG_DIR)/Microsoft.$1.Windows.Bundle.$2.zip: $($(1)_NUGET_TARGETS) $($(1)_WINDOWS_NUGET_TARGETS) $(WORKLOAD_TARGETS) Makefile $(REF_PACK_$(4)) $(SDK_PACK_$(4)) $(SDK_PACK_$(4)_WINDOWS) | $(TMP_PKG_DIR)
	$$(Q) rm -rf $$@ $$@.tmpdir $$@.tmp
	$$(Q) mkdir -p $$@.tmpdir/dotnet/sdk-manifests/$(DOTNET6_VERSION_BAND)/
	$$(Q) mkdir -p $$@.tmpdir/dotnet/packs/Microsoft.$1.Sdk
	$$(Q) mkdir -p $$@.tmpdir/dotnet/packs/Microsoft.$1.Windows.Sdk
	$$(Q) mkdir -p $$@.tmpdir/dotnet/packs/Microsoft.$1.Ref
	$$(Q) $(CP) -r Microsoft.NET.Workload.$1 $$@.tmpdir/dotnet/sdk-manifests/$(DOTNET6_VERSION_BAND)/
	$$(Q) $(CP) -r $(DOTNET_DESTDIR)/Microsoft.$1.Sdk $$@.tmpdir/dotnet/packs/Microsoft.$1.Sdk/$2
	$$(Q) $(CP) -r $(DOTNET_DESTDIR)/Microsoft.$1.Windows.Sdk $$@.tmpdir/dotnet/packs/Microsoft.$1.Windows.Sdk/$2
	$$(Q) $(CP) -r $(DOTNET_DESTDIR)/Microsoft.$1.Ref $$@.tmpdir/dotnet/packs/Microsoft.$1.Ref/$2
	$$(Q_GEN) cd $$@.tmpdir && zip -9rq $$(abspath $$@.tmp) .
	$$(Q) mv $$@.tmp $$@
	$$(Q) echo Created $$@

PACKAGE_TARGETS += $(DOTNET_PKG_DIR)/Microsoft.$1.Windows.Bundle.$2.zip
endef
$(foreach platform,$(DOTNET_WINDOWS_PLATFORMS),$(eval $(call CreateWindowsBundle,$(platform),$($(platform)_NUGET_VERSION_NO_METADATA),$(shell echo $(platform) | tr A-Z a-z),$(shell echo $(platform) | tr a-z A-Z))))

define CreateMsi
$(TMP_PKG_DIR)/Microsoft.NET.Workload.$1.$2.wsx: ./generate-wix.csharp Makefile $(TMP_PKG_DIR)/Microsoft.$1.Windows.Bundle.$2.zip
	$$(Q_GEN) ./generate-wix.csharp "$1" "$$@" "$(TMP_PKG_DIR)/Microsoft.$1.Windows.Bundle.$2.zip.tmpdir/dotnet" "$2"

$(TMP_PKG_DIR)/Microsoft.NET.Workload.$1.$2.msi: $(TMP_PKG_DIR)/Microsoft.NET.Workload.$1.$2.wsx .stamp-check-wixl
	$$(Q_GEN) wixl -o "$$@" "$$<" -a x64

MSI_TARGETS += $(DOTNET_PKG_DIR)/Microsoft.NET.Workload.$1.$2.msi
endef
$(foreach platform,$(DOTNET_WINDOWS_PLATFORMS),$(eval $(call CreateMsi,$(platform),$($(platform)_NUGET_VERSION_NO_METADATA))))

.stamp-check-wixl:
	$(Q) if ! type wixl; then \
		echo "Installing msitools to get wixl..."; \
		if ! brew install msitools; then \
			if ! type wixl; then \
				echo "Failed to install wixl"; \
				exit 1; \
			fi; \
		fi; \
		echo "Installed msitools"; \
	fi
	$(Q) touch $@

TARGETS += $(WORKLOAD_TARGETS)

msi: $(MSI_TARGETS)
package: $(PACKAGE_TARGETS) $(MSI_TARGETS)

ifdef ENABLE_DOTNET
all-local:: $(TARGETS)
endif

# This tells NuGet to use the exact same dotnet version we've configured in Make.config
global.json: $(TOP)/global6.json
	$(CP) $< $@

clean-local::
	$(Q) rm -Rf $(DOTNET_NUPKG_DIR) $(DOTNET_FEED_DIR)
	$(Q) git clean -xfdq

.SECONDARY:
