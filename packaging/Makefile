.PHONY: docker-push

VERSION            ?= $(shell git rev-parse HEAD)
REGISTRY           ?= registry-write.opensource.zalan.do/pathfinder
IMAGE              ?= $(REGISTRY)/skipper:$(VERSION)
LIGHTSTEP_IMAGE    ?= $(REGISTRY)/skipper-lightstep:$(VERSION)
OPENTRACING_IMAGE    ?= $(REGISTRY)/skipper-opentracing:$(VERSION)
ALPINE_BUILD_IMAGE ?= $(REGISTRY)/skipper-alpine-build:latest
PACKAGE            ?= github.com/zalando/skipper
CGO_ENABLED        ?= 0
GO111              ?= on

default: docker-build

skipper:
	GO111MODULE=$(GO111) GOOS=linux GOARCH=amd64 CGO_ENABLED=$(CGO_ENABLED) go build -o skipper ../cmd/skipper/*.go

eskip:
	GO111MODULE=$(GO111) GOOS=linux GOARCH=amd64 CGO_ENABLED=$(CGO_ENABLED) go build -o eskip ../cmd/eskip/*.go

plugins:
	/bin/ls -1 ../opentracingplugin/tracers/ | grep -v basic | while read f; do GO111MODULE=$(GO111) go build -buildmode=plugin -o build/tracing_$$f.so ../opentracingplugin/tracers/$$f/*.go; done

clean:
	rm -rf skipper eskip build/

docker-build: clean skipper eskip
	docker build -t $(IMAGE) .

docker-push:
	docker push $(IMAGE)

alpine.build: docker-alpine-build
	docker run --rm \
		-v $(shell pwd)/..:/build \
		-w /build/packaging $(ALPINE_BUILD_IMAGE) \
		make skipper eskip

alpine.plugins: docker-alpine-build
	docker run --rm -v $(shell pwd)/..:/build \
		-w /build/packaging $(ALPINE_BUILD_IMAGE) \
		make plugins

docker-alpine-build:
	docker build --rm -t $(ALPINE_BUILD_IMAGE) -f Dockerfile.alpine .

docker-lightstep-build: alpine.build alpine.plugins
	docker build -t $(LIGHTSTEP_IMAGE) -f Dockerfile.lightstep .

docker-lightstep-push:
	docker push $(LIGHTSTEP_IMAGE)

docker-opentracing-build: alpine.build alpine.plugins
	docker build -t $(OPENTRACING_IMAGE) -f Dockerfile.opentracing .

docker-opentracing-push:
	docker push $(OPENTRACING_IMAGE)