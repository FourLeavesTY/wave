WAVE_ML_VERSION := v0.8.0
# HACK: Linux uses GNU sed, while OSX uses BSD - need to install gsed to unify.
SED=$(shell command -v gsed || command -v sed)

all: build ## Build h2o_wave wheel

.PHONY: build
build:
	$(MAKE) build-wave
	$(MAKE) build-lightwave
	$(MAKE) build-lightwave-web

.PHONY: build-wave
build-wave: 
	cd h2o_wave && $(MAKE) build

.PHONY: build-lightwave
build-lightwave: 
	cd h2o_lightwave && $(MAKE) build

.PHONY: build-lightwave-web
build-lightwave-web: 
	cd h2o_lightwave_web && $(MAKE) build

setup: ## Install dependencies
	git clone --depth 1 --branch $(WAVE_ML_VERSION) https://github.com/h2oai/wave-ml.git h2o_wave_ml
	python3 -m venv venv
	./venv/bin/python -m pip install --upgrade pip
	./venv/bin/python -m pip install setuptools wheel httpx uvicorn starlette pdoc3 flake8
	./venv/bin/python -m pip install h2o_wave_ml/
	# TODO examples pip install is wasteful for CI
	./venv/bin/python -m pip install -r examples/requirements.txt
	./venv/bin/python -m pip install --editable h2o_wave
	[ -d ../ui/build ] && $(MAKE) build-lightwave-web || echo "WARN: ../ui/build doesn't exist, skipping build-lightwave-web"
	./venv/bin/python -m pip install --editable h2o_lightwave
	./venv/bin/python -m pip install --editable h2o_lightwave_web
	rm -f h2o_wave/h2o_wave/metadata.py
	echo "# Generated in setup.py\n__platform__ = 'linux'\n__arch__ = 'amd64'" > h2o_wave/h2o_wave/metadata.py

setup-tests: ## Install dependencies for tests only
	python3 -m venv venv
	./venv/bin/python -m pip install --editable h2o_wave
	./venv/bin/pip install httpx

.PHONY: docs
docs: ## Build API docs
	./venv/bin/pdoc --force  --template-dir docs/templates --output-dir build/docs h2o_wave
	./venv/bin/pdoc --force  --template-dir docs/templates --output-dir build/docs h2o_wave_ml/h2o_wave_ml
	mkdir -p ../website/docs/api/h2o_wave_ml
	mv build/docs/h2o_wave/* ../website/docs/api/
	mv build/docs/h2o_wave_ml/* ../website/docs/api/h2o_wave_ml/
	./venv/bin/python sync_examples.py

test:
	./venv/bin/python -m tests
	echo "Testing using BASE_URL" && H2O_WAVE_BASE_URL="/foo/" ./venv/bin/python -m tests
	echo "Testing using LOCAL UPLOAD" && H2O_WAVE_WAVED_DIR=".." ./venv/bin/python -m tests
	echo "Testing using LOCAL UPLOAD AND BASE URL" && H2O_WAVE_WAVED_DIR=".." H2O_WAVE_BASE_URL="/foo/" ./venv/bin/python -m tests

clean:
	rm -rf docs/build venv h2o_wave_ml
	cd h2o_wave && $(MAKE) purge
	cd h2o_lightwave && $(MAKE) purge
	cd h2o_lightwave_web && $(MAKE) purge

.PHONY: tag
tag: # Bump version
	$(SED) -i -r -e "s/__version__.+/__version__ = '$(VERSION)'/" h2o_wave/h2o_wave/version.py
	$(SED) -i -r -e "s/__version__.+/__version__ = '$(VERSION)'/" h2o_lightwave/h2o_lightwave/version.py
	$(SED) -i -r -e "s/__version__.+/__version__ = '$(VERSION)'/" h2o_lightwave_web/h2o_lightwave_web/version.py

help: ## List all make tasks
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'

