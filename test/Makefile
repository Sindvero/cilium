# Copyright Authors of Cilium
# SPDX-License-Identifier: Apache-2.0

include ../Makefile.defs

# Determines if the eBPF unit tests are ran with `sudo`
RUN_WITH_SUDO ?= true
ifeq ($(RUN_WITH_SUDO), true)
	RUN_WITH_SUDO=-exec sudo
else
	RUN_WITH_SUDO=
endif

PROVISION ?= true
# If you set provision to false the test will run without bootstrapping the
# cluster again.
HOLD_ENVIRONMENT ?= true

TEST_ARTIFACTS = ./tmp.yaml ./*_service_manifest.json ./*_manifest.yaml
TEST_ARTIFACTS += ./*_policy.json ./k8s-*.xml ./runtime.xml ./test_results
TEST_ARTIFACTS += ./test.test

NETNEXT ?= false
KUBEPROXY ?= 1
NO_CILIUM_ON_NODES ?= ""

GINKGO = $(QUIET) ginkgo

REGISTRY_CREDENTIALS ?= "${DOCKER_LOGIN}:${DOCKER_PASSWORD}"

.PHONY = all build test run k8s-test k8s-kind clean

all: build

build:
	@$(ECHO_GINKGO)$@
	$(GINKGO) build

build-darwin:
	@$(ECHO_GINKGO)$@
	GOOS=darwin $(GINKGO) build

test: run k8s-test

run:
	INTEGRATION_TESTS=true KERNEL=net-next ginkgo --focus "Runtime" -v -- --cilium.provision=$(PROVISION) --cilium.registryCredentials=$(REGISTRY_CREDENTIALS)

k8s-test:
	INTEGRATION_TESTS=true ginkgo --focus "K8s" -v -- --cilium.provision=$(PROVISION) --cilium.registryCredentials=$(REGISTRY_CREDENTIALS)

# Match kind-image target in parent directory
k8s-kind: export DOCKER_REGISTRY=localhost:5000
k8s-kind:
	@if [ -z "$(FOCUS)" ]; then \
		>&2 echo "usage: FOCUS=K8sFoo make k8s-kind"; \
		exit 1; \
	fi
	@CNI_INTEGRATION=kind \
		K8S_VERSION="$$(kubectl version -o json | jq -r '.serverVersion | "\(.major).\(.minor)"')" \
		NETNEXT="$(NETNEXT)" \
		KUBEPROXY="$(KUBEPROXY)" \
		NO_CILIUM_ON_NODES="$(NO_CILIUM_ON_NODES)" \
		INTEGRATION_TESTS=true ginkgo --focus "$(FOCUS)" -v -- \
			-cilium.testScope=k8s \
			-cilium.provision=false \
			-cilium.kubeconfig=$$(echo ~/.kube/config) \
			-cilium.passCLIEnvironment=true \
			-cilium.image="$(DOCKER_REGISTRY)/cilium/cilium-dev" \
			-cilium.tag="local" \
			-cilium.operator-image="quay.io/cilium/operator" \
			-cilium.operator-suffix="-ci" \
			-cilium.holdEnvironment=$(HOLD_ENVIRONMENT)

clean:
	@$(ECHO_CLEAN)
	-$(QUIET) rm -rf $(TEST_ARTIFACTS)
