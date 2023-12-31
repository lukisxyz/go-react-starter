.PHONY: clean critic security lint test build run

APP_NAME = apiserver
BUILD_DIR = $(PWD)/build

clean:
	rm -rf ./build

critic:
	gocritic check -enableAll ./...

security:
	bin/gosec ./...

lint:
	bin/golangci-lint run ./...

test: clean critic security lint
	go test -v -timeout 30s -coverprofile=cover.out -cover ./...
	go tool cover -func=cover.out

build: test
	CGO_ENABLED=0 go build -ldflags="-w -s" -o $(BUILD_DIR)/$(APP_NAME) main.go

run: build
	bin/air

docker.run: docker.network docker.chi

docker.network:
	docker network inspect dev-network >/dev/null 2>&1 || \
	docker network create -d bridge dev-network

docker.chi.build:
	docker build -t chi .

docker.chi: docker.chi.build
	docker run --rm -d \
		--name cgapp-chi \
		--network dev-network \
		-p 5000:5000 \
		chi

docker.stop: docker.stop.chi

docker.stop.chi:
	docker stop cgapp-chi

cmgr:
	bin/migrate create -ext sql -dir database/migrations -seq ${name}

migup:
	bin/migrate -path database/migrations -database "${PSQL}" -verbose up

migdown:
	bin/migrate -path database/migrations -database "${PSQL}" -verbose down

setupair:
	curl -sSfL https://raw.githubusercontent.com/cosmtrek/air/master/install.sh | sh -s -- -b bin/

setupmigrate:
	curl -L https://github.com/golang-migrate/migrate/releases/download/v4.16.2/migrate.linux-amd64.tar.gz | tar xvz -C bin/

setupgosec:
	curl -sfL https://raw.githubusercontent.com/securego/gosec/master/install.sh | sh -s -- -b bin/ v2.16.0

setupgocritic:
	curl -L https://github.com/go-critic/go-critic/archive/refs/tags/v0.9.0.tar.gz | tar xvz -C bin/

setuplint:
	curl -sSfL https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s -- -b bin/ v1.54.1

setup: setupair setupmigrate setupgosec setupgocritic setuplint