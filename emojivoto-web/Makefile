svc_name := emojivoto-web

WEB_PORT ?= 8080
VOTINGSVC_HOST ?= localhost:8081
EMOJISVC_HOST ?= localhost:8082
WEBPACK_DEV_SERVER ?= http://localhost:8083
INDEX_BUNDLE ?= webapp/dist/index_bundle.js

K8S_VOTINGSVC_HOST ?= voting-svc.emojivoto-dev:8080
K8S_EMOJISVC_HOST ?= emoji-svc.emojivoto-dev:8080

all: clean protoc webpack package-web test package

include ../common.mk

webpack:
	$(MAKE) -C webapp

package-web: webpack
	mkdir -p $(target_dir)/web
	cp web/favicon.ico $(target_dir)/web
	cp -a webapp/dist $(target_dir)

compile-vote-bot:
	GOOS=linux go build -v -o $(target_dir)/emojivoto-vote-bot cmd/vote-bot/main.go

compile: compile-vote-bot

run-dev:
	WEB_PORT=$(WEB_PORT) VOTINGSVC_HOST=$(VOTINGSVC_HOST) EMOJISVC_HOST=$(EMOJISVC_HOST) WEBPACK_DEV_SERVER=$(WEBPACK_DEV_SERVER) go run cmd/server.go

run-tel:
	WEB_PORT=$(WEB_PORT) VOTINGSVC_HOST=$(K8S_VOTINGSVC_HOST) EMOJISVC_HOST=$(K8S_EMOJISVC_HOST) WEBPACK_DEV_SERVER=$(WEBPACK_DEV_SERVER) go run cmd/server.go
