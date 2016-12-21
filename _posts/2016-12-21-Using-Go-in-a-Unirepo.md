---
layout: post
title:  "Using Go in a Unirepo"
date:   2016-12-21 12:11:28
categories: golang
---

I've recently started a love affair with Go, much to the disgust of some of my coworkers. One of the best things about using Go is how ridiculously fast your work cycle becomes when compared to other languages, such as Erlang and Elixir. Simple things that require a tremendous amount of work are a non-issue in Go, which ultimately helps you get to market faster.

However, a common complaint about Go is package and dependency management. There are no shortages of community developed package managers for Go, but in general I don't think a package manager is really needed so long as you structure your work correctly.

### Unirepo Folder Layout
Short and simple, here's what works the best:

	code/ <--- This is your GOPATH, but you don't ever need to set it. Keep reading.
	  bin/
	  pkg/
	  src/
	    make/
	      Makefile
	  tools/
	  .git/
	  .gitignore

### Installing Dependencies
Again, keep it simple and just do

	GOPATH=/path/to/code go get -u github.com/...

### Setting Up Your Code
All new projects should live in `code/src/<project_name>`. In each project folder, create a Makefile and include the following in it

	include ../make/Makefile

### Compiling Your Code
This one is a bit tricky, but with the magic of Bash and Make, you can make life pretty easy. In `code/src/make/Makefile`, paste the following

	export GOPATH = $(realpath $(shell dirname $(realpath $(lastword $(MAKEFILE_LIST))))/../../)
	export GOBIN = ${GOPATH}/bin
	export APP = $(shell basename "$$PWD")

	build::
		go build
		go install

	run::
		../../bin/$(APP)

	test::
		@mkdir -p $(GOPATH)/tests/
		@echo 'mode: set' > coverage.txt && go list $(APP)/... | xargs -n1 -I{} sh -c 'go test -covermode=set 	-coverprofile=coverage.tmp {} && tail -n +2 coverage.tmp >> coverage.txt' && rm coverage.tmp
		@go tool cover -html=coverage.txt -o $(GOPATH)/tests/$(APP).html
		@rm coverage.txt

	package::
		GOOS=linux GOARCH=amd64 go build -o ../../bin/$(APP).linux
		GOOS=darwin GOARCH=amd64 go build -o ../../bin/$(APP).mac

	%::
			@:
	.PHONY: test

Now any project that includes this file (as noted in the section above) will work with Make. For example

	cd code/src/someProject
	make
	make run
	make test

It's important to note this setup **automatically sets your GOPATH, you do not need to set it in your bash profile, and is 100% portable across folder locations and OS's.**
### This Is Dumb; Why Would You Do This?
Speed and simplicity of management.

Consider working in a large environment where many people will write many different libraries. Every time you want to make a change to a library, you need to commit your code in some other repository, push that code, pull it back down to your Go project, compile, etc. If you have a bug in your new code, you'll have to do it all over again. Making many small changes? Good luck.

With this setup, you commit the whole tree, and just focus on what you need to do: edit your code. Need to make a change to a dep? Then do it and recompile. The next time you push and merge, your changes will be available to every app within your organization.

As an added bonus, the next time Github is DDOS'd, your work flow won't be disrupted, assuming you're not using Github as your primary repository (you're not, right?).

Enjoy. 