DOCKER  = docker
PACKAGE ?= nickolasburr/magento-php
VERSION ?= 7.2-fpm
TAGNAME ?= "$(PACKAGE):$(VERSION)"

.PHONY: help build

help:
	@echo "USAGE:\n\nmake <COMMAND> [OPTIONS]"

build:
	$(DOCKER) build -t $(TAGNAME) .
