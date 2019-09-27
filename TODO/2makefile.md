https://www.cnblogs.com/aoyihuashao/archive/2010/01/18/1650865.html



https://blog.csdn.net/ruglcc/article/details/7814546





http://www.ruanyifeng.com/blog/2015/02/make.html





















```makefile
.SILENT :
.PHONY : dep vet clean dist package test

NAME := cistern
PRE := oc
ROOF := fhyx.tech/oceans/$(NAME)

WITH_ENV = env `cat .env 2>/dev/null | xargs`
UNAME_S := $(shell uname -s)
ifeq ($(UNAME_S),Darwin)
    HOST := $(shell scutil --get LocalHostName)
else
    HOST := $(shell hostname)
endif

DATE := $(shell date '+%Y%m%dT%H%M')
STAMP := $(shell date +%s)
USER := $(shell echo ${USER})
TAG:=$(shell git describe --tags --always)
LDFLAGS:=-X $(ROOF)/settings.Name=$(NAME) -X $(ROOF)/cmd.version=$(TAG) -X $(ROOF)/cmd.built=$(DATE) -X $(ROOF)/cmd.buildStamp=$(STAMP) -X $(ROOF)/cmd.buildUser=$(USER) -X $(ROOF)/cmd.buildHost=$(HOST)

COMMANDS = vet clean dist
.PHONY: $(COMMANDS)

main:
	echo "Building $(NAME)"
	go build -ldflags "$(LDFLAGS)" .

help:
	@echo "commands: $(COMMANDS)"

all: clean $(COMMANDS)

vet:
	echo "Checking ."
	go vet -vettool=$(which shadow) -atomic -bool -copylocks -nilfunc -printf -rangeloops -unreachable -unsafeptr -unusedresult ./...

clean:
	echo "Cleaning dist"
	rm -rf dist
	rm -f $(NAME) $(NAME)-*

dist/linux_amd64/$(NAME): $(SOURCES)
	echo "Building $(NAME) of linux"
	mkdir -p dist/linux_amd64 && GOOS=linux GOARCH=amd64 go build -ldflags "$(LDFLAGS) -s -w" -o dist/linux_amd64/$(PRE)-$(NAME) $(ROOF)

dist/darwin_amd64/$(NAME): $(SOURCES)
	echo "Building $(NAME) of darwin"
	mkdir -p dist/darwin_amd64 && GOOS=darwin GOARCH=amd64 go build -ldflags "$(LDFLAGS) -w" -o dist/darwin_amd64/$(PRE)-$(NAME) $(ROOF)

dist: clean dist/linux_amd64/$(NAME) dist/darwin_amd64/$(NAME)

package: dist
	echo "Packaging $(NAME)"
	ls dist/linux_amd64 | xargs tar -cvJf $(NAME)-linux-amd64-$(TAG).tar.xz -C dist/linux_amd64

.PHONY: binary-deploy
binary-deploy:
	@echo "copy binary to earth"
	@scp dist/linux_amd64/??-* earth:dist/linux_amd64/

.PHONY: package-upload
package-upload:
	@echo "copy package.tar.?z to venus"
	@scp *-linux-amd64-*.tar.?z gopkg:/var/www/gopkg/

docker-build: dist/linux_amd64/$(NAME)
	echo "Building docker image"
	cp -rf Dockerfile* dist/
	docker build -t fhyx/cistern:$(TAG) dist/
	docker tag fhyx/cistern:$(TAG) fhyx/cistern:latest
.PHONY: $@


```









```go
package cmd

import (
	"fmt"
	"runtime"

	"github.com/spf13/cobra"
)

var (
	version   = "dev"
	built     = "N/A"
	buildUser = "None"
	name      = "cistern"
)

var versionCmd = &cobra.Command{
	Use:   "version",
	Short: "Print the version number",
	Long:  ``,
	Run: func(cmd *cobra.Command, args []string) {
		fmt.Printf("%s %s built %s by %s (%s %s-%s)\n", name, version, built, buildUser, runtime.Version(), runtime.GOOS, runtime.GOARCH)
	},
}

func init() {
	RootCmd.AddCommand(versionCmd)
}

func inDevelop() bool {
	return version == "dev"
}
```

