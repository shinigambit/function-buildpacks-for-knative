RULES.MK ?= ../../../rules.mk
include $(RULES.MK)

path ?= .
template-ce.path := $(abspath $(path))

$(eval $(call INCLUDE_FILE, $(ROOT_DIR)/builder))

template-ce.image := kn-fn-test/template-ce
template-ce.image_paths := $(shell find $(template-ce.path) -mindepth 1 -maxdepth 1 -type d)
$(template-ce.image_paths): $(PACK) $(builder.build)
	cd $@ && $(PACK) build $(template-ce.image):$(notdir $@) --builder $(shell cat $(builder.build))

template-ce.clean := $(addsuffix .clean,$(template-ce.image_paths))
$(template-ce.clean):
	-docker rmi -f $(template-ce.image):$(basename $(notdir $@))

.PHONY: template-tests.images
template-tests.images .PHONY: $(template-ce.image_paths)
clean .PHONY: $(template-ce.clean)
