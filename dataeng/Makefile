PACKAGE := dataeng

CONTAINER := dataeng
REGISTRY := europe-north1-docker.pkg.dev/vivid-env-450617-g5/teemu-images/
TAG ?= latest

# Get version from package __init__ file:
VERSION := $(shell sed -ne "/__version__/s/^.*=.'\(.*\)'$$/\1/p" $(PACKAGE)/__init__.py)

# ---- Generic rules to build containers and publish them
# .PHONY: all
# all: build docker

# .PHONY: build
# build: run-tests

# .PHONY: docker
# docker: run-tests version.json
# 	docker buildx build --secret id=gcp_secret,src=$(HOME)/.config/gcloud/application_default_credentials.json --platform linux/amd64 -t  $(CONTAINER) --pull=true .
# 	rm -rf version.json

# .PHONY: git-tag-and-push
# git-tag-and-push:
# 	git tag -a $(VERSION) -m v$(VERSION) && git push origin $(VERSION)

