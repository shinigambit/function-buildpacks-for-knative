RULES.MK ?= ../../rules.mk
include $(RULES.MK)

path ?= .
python-buildpack.path := $(abspath $(path))

include $(python-buildpack.path)/../rules.mk

$(eval $(call VERSION_template,python-buildpack,$(python-buildpack.path)))
python-buildpack.version := $(python-buildpack.version.release)

python-buildpack.go-sources := \
	$(python-buildpack.path)/go.mod \
	$(python-buildpack.path)/go.sum \
	$(shell find '$(python-buildpack.path)' -type f -iname '*.go' -not -path '$(python-buildpack.path)/tests/*') \
	$(shell find '$(python-buildpack.path)' -type d -not -path '$(python-buildpack.path)/tests' -mindepth 1) \

python-buildpack.test-sources := \
	$(python-buildpack.path)/go.mod \
	$(python-buildpack.path)/go.sum \
	$(shell find '$(python-buildpack.path)/tests' -type f -iname '*.go') \
	$(python-buildpack.path)/tests \

python-buildpack.config-sources := $(python-buildpack.path)/buildpack.toml

python-buildpack.image := $(registry)/python-buildpack:$(python-buildpack.version)
python-buildpack.image_with_deps := $(registry)/python-buildpack-with-deps:$(python-buildpack.version)

python-buildpack.build-bin := $(python-buildpack.path)/bin/build
python-buildpack.detect-bin := $(python-buildpack.path)/bin/detect
python-buildpack.main-bin := $(abspath $(python-buildpack.path)/bin/main)
python-buildpack.bin := $(python-buildpack.build-bin) $(python-buildpack.detect-bin)

python-buildpack.deps := $(python-buildpack.bin) $(python-buildpack.config-sources)

$(python-buildpack.build-bin) $(python-buildpack.detect-bin): $(python-buildpack.main-bin)
	@mkdir -p $(@D)
	ln -sf $(notdir $(python-buildpack.main-bin)) $@

$(python-buildpack.main-bin): $(python-buildpack.go-sources)
	@mkdir -p $(@D)
	cd $(python-buildpack.path) && GOOS="linux" go build -ldflags='-s -w' -o $@ kn-fn/python-function-buildpack/cmd/main

python-buildpack.bin: $(python-buildpack.bin)

python-buildpack.package-deps := $(PACK) 
python-buildpack.package.online := $(buildpack.build_dir)/packages/python/$(python-buildpack.version)/online
python-buildpack.package.offline := $(buildpack.build_dir)/packages/python/$(python-buildpack.version)/offline
python-buildpack.online.out := $(buildpack.out_dir)/python/online/python-buildpack-$(python-buildpack.version).image
python-buildpack.offline.out := $(buildpack.out_dir)/python/offline/python-buildpack-with-deps-$(python-buildpack.version).image

$(python-buildpack.package.online)/package.toml $(python-buildpack.package.offline)/package.toml: %/package.toml:
	printf '[buildpack]\nuri = "%s"\n\n[platform]\nos = "%s"\n' $* "linux" > $@

$(python-buildpack.package.online): $(CREATE-PACKAGE) $(python-buildpack.deps)
	@mkdir -p $(@D)
	$(CREATE-PACKAGE) \
		--cache-location "$(buildpack.build_dir)/cache/carton-cache" \
		--destination "$@" \
		--source "$(python-buildpack.path)" \
		--version $(python-buildpack.version)

python-buildpack.online.build := $(buildpack.build_dir)/python/online/python-buildpack-$(python-buildpack.version).image
$(python-buildpack.online.build): $(python-buildpack.package.online) $(PACK) $(python-buildpack.package.online)/package.toml
	@mkdir -p $(@D)
	$(PACK) buildpack package $(python-buildpack.image) --config $(python-buildpack.package.online)/package.toml
	printf '$(python-buildpack.image)' > $@

$(python-buildpack.online.out): $(python-buildpack.online.build)
	@mkdir -p $(@D)
	cp $< $@

python-buildpack.online: $(python-buildpack.online.out) $(python-buildpack.online.out).sha256

$(python-buildpack.package.offline): $(CREATE-PACKAGE) $(python-buildpack.deps)
	@mkdir -p $(@D)
	$(CREATE-PACKAGE) \
		--cache-location "$(buildpack.build_dir)/cache/carton-cache" \
		--destination "$@" \
		--source "$(python-buildpack.path)" \
		--include-dependencies \
		--version $(python-buildpack.version)

python-buildpack.offline.build := $(buildpack.build_dir)/python/offline/python-buildpack-$(python-buildpack.version).image
$(python-buildpack.offline.build): $(python-buildpack.package.offline) $(PACK) $(python-buildpack.package.offline)/package.toml
	@mkdir -p $(@D)
	$(PACK) buildpack package $(python-buildpack.image_with_deps) --config $(python-buildpack.package.offline)/package.toml
	printf '$(python-buildpack.image_with_deps)' > $@

$(python-buildpack.offline.out): $(python-buildpack.offline.build)
	@mkdir -p $(@D)
	cp $< $@

python-buildpack.offline: $(python-buildpack.offline.out) $(python-buildpack.offline.out).sha256

python-buildpack.package: python-buildpack.online python-buildpack.offline

python-buildpack.publish: python-buildpack.package
	docker push $(python-buildpack.image)
	docker push $(python-buildpack.image_with_deps)

python-buildpack.tests: $(python-buildpack.go-sources) $(python-buildpack.test-sources)
	cd $(python-buildpack.path) && go test -v -count=1 -timeout 30s kn-fn/python-function-buildpack/tests

python-buildpack.clean:
	rm -rf $(python-buildpack.path)/out
	rm -rf $(python-buildpack.path)/bin
	rm -rf $(python-buildpack.path)/package.toml

all buildpacks .PHONY: python-buildpack.package
buildpacks.offline .PHONY: python-buildpack.offline
buildpacks.online .PHONY: python-buildpack.online
publish-buildpacks .PHONY: python-buildpack.publish
tests buildpack-tests .PHONY: python-buildpack.tests
clean .PHONY: python-buildpack.clean
