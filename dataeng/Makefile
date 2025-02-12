PACKAGE := cmapi

CONTAINER := vnoc/automate-$(PACKAGE)
REGISTRY := valkama.saunalahti.fi
TAG ?= latest

# Get version from package __init__ file:
VERSION := $(shell sed -ne "/__version__/s/^.*=.'\(.*\)'$$/\1/p" $(PACKAGE)/__init__.py)

# ---- Generic rules to build containers and publish them
.PHONY: all
all: build docker

.PHONY: build
build: run-tests

.PHONY: docker
docker: run-tests version.json
	docker buildx build --secret id=gcp_secret,src=$(HOME)/.config/gcloud/application_default_credentials.json --platform linux/amd64 -t  $(CONTAINER) --pull=true .
	rm -rf version.json

version.json:
	@echo "{\"container\": \"$(CONTAINER)\", \
			\"registry\": \"$(REGISTRY)\", \
			\"repository\": \"$(shell git config remote.origin.url)\", \
			\"commit\": \"$(shell git rev-parse HEAD)\", \
			\"date\": \"$(shell date +"%Y%m%dT%H%M")\", \
			\"by\": {\
				\"name\": \"$(shell git config user.name)\", \
				\"email\": \"$(shell git config user.email)\"\
			}}" | python3 -m json.tool | tee $@

.PHONY: clean
clean:
	git clean -Xdf -e '!*.iml'

.PHONY: re
re: clean all

# project specific makefile rules
.built:
	pipenv sync --dev
	touch .built

.PHONY: run-tests
run-tests: .built
	pipenv run python3 -X dev -m pytest -rs --junitxml=pytests.xml --cov-branch --cov-report=term --cov-report=xml:coverage.xml --cov-report=html --cov=$(PACKAGE) tests

.PHONY: pycodestyle
pycodestyle: .built
	pipenv run pycodestyle --statistics --$(shell grep 'max-line-length=' .pylintrc) $(PACKAGE)

.PHONY: vulture
vulture: .built
	pipenv run vulture --min-confidence=90 $(PACKAGE)

.PHONY: pylintchecker
pylintchecker: .built
	pipenv run pylint --rcfile=.pylintrc --fail-under=9.7 $(PACKAGE)

.PHONY: mypy
mypy: .built
	pipenv run mypy --ignore-missing-imports -i $(PACKAGE);

.PHONY: lint
lint: pylintchecker vulture pycodestyle mypy

.PHONY: deps-check
deps-check:
	mkdir -p reports && dependency-check --cveValidForHours 24 --out reports --format ALL \
		--enableExperimental --disableAssembly --disableBundleAudit --disableNodeJS --disableOssIndex \
		--scan $(PACKAGE) --project $(PACKAGE)

.PHONY: beautify
beautify: .built
	pipenv run yapf --in-place --parallel -vv --recursive --style='{based_on_style: google, column_limit:120, allow_split_before_dict_value: false}' $(PACKAGE) tests

.PHONY: git-tag-test
git-tag-test:
	git tag $(VERSION)

.PHONY: git-tag-and-push
git-tag-and-push:
	git tag -a $(VERSION) -m v$(VERSION) && git push origin $(VERSION)

.PHONY: valkama
valkama: docker
	@echo "Tagging '$(CONTAINER)' as '$(TAG)' and pushing into $(REGISTRY)"
	docker tag $(CONTAINER) $(REGISTRY)/$(CONTAINER):$(TAG)
	docker push $(REGISTRY)/$(CONTAINER):$(TAG) || echo "hint: use 'docker login $(REGISTRY)' to authenticate"
