# Use one of these commands to build the manifest for Bash:
#
# - make
# - make DEBUG=1
# - make SGX=1
# - make SGX=1 DEBUG=1
#
# Use `make clean` to remove Graphene-generated files.

# Relative path to Graphene root and key for enclave signing
GRAPHENEDIR ?= ../..
SGX_SIGNER_KEY ?= $(GRAPHENEDIR)/Pal/src/host/Linux-SGX/signer/enclave-key.pem

ifeq ($(DEBUG),1)
GRAPHENE_LOG_LEVEL = debug
else
GRAPHENE_LOG_LEVEL = error
endif

.PHONY: all
all: bash.manifest
ifeq ($(SGX),1)
all: bash.manifest.sgx bash.sig bash.token
endif

include ../../Scripts/Makefile.configs

bash.manifest: manifest.template
	graphene-manifest \
		-Dlog_level=$(GRAPHENE_LOG_LEVEL) \
		-Dexecdir=$(shell dirname $(shell which bash)) \
		-Darch_libdir=$(ARCH_LIBDIR) \
		$< >$@

# Generating the SGX-specific manifest (*.manifest.sgx), the enclave signature,
# and the token for enclave initialization.
bash.manifest.sgx: bash.manifest
	graphene-sgx-sign \
		--key $(SGX_SIGNER_KEY) \
		--manifest bash.manifest \
		--output $@

bash.sig: bash.manifest.sgx

bash.token: bash.sig
	graphene-sgx-get-token --output bash.token --sig bash.sig

ifeq ($(SGX),)
GRAPHENE = graphene-direct
else
GRAPHENE = graphene-sgx
endif

.PHONY: regression
regression: all
	@mkdir -p scripts/testdir

	$(GRAPHENE) ./bash -c "ls" > OUTPUT
	@grep -q "Makefile" OUTPUT && echo "[ Success 1/7 ]"
	@rm OUTPUT

	$(GRAPHENE) ./bash -c "cd scripts && bash bash_test.sh 1" > OUTPUT
	@grep -q "hello 1" OUTPUT      && echo "[ Success 2/7 ]"
	@grep -q "createdfile" OUTPUT  && echo "[ Success 3/7 ]"
	@grep -q "somefile" OUTPUT     && echo "[ Success 4/7 ]"
	@grep -q "current date" OUTPUT && echo "[ Success 5/7 ]"
	@rm OUTPUT

	$(GRAPHENE) ./bash -c "cd scripts && bash bash_test.sh 3" > OUTPUT
	@grep -q "hello 3" OUTPUT      && echo "[ Success 6/7 ]"
	@rm OUTPUT

	$(GRAPHENE) ./bash -c "readlink /proc/self/exe" > OUTPUT
	@grep -qE "^(/usr)?/bin/readlink" OUTPUT && echo "[ Success 7/7 ]"
	@rm OUTPUT

	@rm -rf scripts/testdir


.PHONY: clean
clean:
	$(RM) *.manifest *.manifest.sgx *.token *.sig OUTPUT scripts/testdir/*

.PHONY: distclean
distclean: clean
