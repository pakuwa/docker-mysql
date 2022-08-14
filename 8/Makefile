-include env_make

MYSQL_VER ?= 8.0.30
MYSQL_VER_MINOR = $(shell echo "${MYSQL_VER}" | grep -oE '^[0-9]+\.[0-9]+')

ALPINE_VER ?= 3.15

NPROC =

PLATFORM ?= linux/amd64

# Remove minor version from tag
TAG ?= $(MYSQL_VER_MINOR)

ifneq ($(STABILITY_TAG),)
    ifneq ($(TAG),latest)
        override TAG := $(TAG)-$(STABILITY_TAG)
    endif
endif

REPO = wawo/mysql
NAME = mysql-$(MYSQL_VER_MINOR)

.PHONY: build buildx-build buildx-push buildx-build-amd64 test push shell run start stop logs clean release

default: build

build:
	docker build -t $(REPO):$(TAG) \
	    --build-arg MYSQL_VER=$(MYSQL_VER) \
	    --build-arg NPROC=$(NPROC) ./

# --load doesn't work with multiple platforms https://github.com/docker/buildx/issues/59
# we need to save cache to run tests first.
buildx-build-amd64:
	docker buildx build --platform linux/amd64 -t $(REPO):$(TAG) \
	    --build-arg MYSQL_VER=$(MYSQL_VER) \
	    --build-arg NPROC=$(NPROC) \
		--load \
	    ./

buildx-build:
	docker buildx build --platform $(PLATFORM) -t $(REPO):$(TAG) \
	    --build-arg MYSQL_VER=$(MYSQL_VER) \
	    --build-arg NPROC=$(NPROC) ./

buildx-push:
	docker buildx build --push --platform $(PLATFORM) -t $(REPO):$(TAG) \
	    --build-arg MYSQL_VER=$(MYSQL_VER) \
	    --build-arg NPROC=$(NPROC) ./

test:
	cd ./tests && IMAGE=$(REPO):$(TAG) ./run.sh

push:
	docker push $(REPO):$(TAG)

shell:
	docker run --rm --name $(NAME) -i -t $(PORTS) $(VOLUMES) $(ENV) $(REPO):$(TAG) /bin/bash

run:
	docker run --rm --name $(NAME) -e DEBUG=1 -e MYSQL_RANDOM_ROOT_PASSWORD=1 $(PORTS) $(VOLUMES) $(ENV) $(REPO):$(TAG) $(CMD)

start:
	docker run -d --name $(NAME) -e MYSQL_RANDOM_ROOT_PASSWORD=1 $(PORTS) $(VOLUMES) $(ENV) $(REPO):$(TAG)

stop:
	docker stop $(NAME)

logs:
	docker logs $(NAME)

clean:
	-docker rm -f $(NAME)

release: build push
