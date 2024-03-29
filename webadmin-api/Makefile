# This Makefile requires GNU Make.
MAKEFLAGS += --silent

# Settings
C_BLU='\033[0;34m'
C_GRN='\033[0;32m'
C_RED='\033[0;31m'
C_YEL='\033[0;33m'
C_END='\033[0m'

include ../.env

DOCKER_ABBR="WEBADMIN_API"
DOCKER_HOST=$(HOSTNAME)
DOCKER_PORT=$(WEBADMIN_API_PORT)
DOCKER_NAME=$(PROJECT)$(WEBADMIN_API_PROJECT)$(PROJECT_CODE)

CURRENT_DIR=$(patsubst %/,%,$(dir $(realpath $(firstword $(MAKEFILE_LIST)))))
DIR_BASENAME=$(shell basename $(CURRENT_DIR))
ROOT_DIR=$(CURRENT_DIR)
DOCKER_COMPOSE?=$(DOCKER_USER) docker compose
DOCKER_COMPOSE_RUN=$(DOCKER_COMPOSE) run --rm
DOCKER_EXEC_TOOLS_APP=$(DOCKER_USER) docker exec -it $(DOCKER_NAME) sh

COMPOSER_INSTALL="composer update"

help: ## shows this Makefile help message
	echo 'usage: make [target]'
	echo
	echo 'targets:'
	egrep '^(.+)\:\ ##\ (.+)' ${MAKEFILE_LIST} | column -t -c 2 -s ':#'

# -------------------------------------------------------------------------------------------------
#  System
# -------------------------------------------------------------------------------------------------
.PHONY: fix-permission host-check

fix-permission: ## sets project directory permission
	$(DOCKER_USER) chown -R ${USER}: $(ROOT_DIR)/

host-check: ## shows this project ports availability on local machine
	echo "Checking configuration for "${C_YEL}"$(DOCKER_NAME)"${C_END}" container:";
	if [ -z "$$($(DOCKER_USER) lsof -i :$(DOCKER_PORT))" ]; then \
		echo ${C_BLU}"$(DOCKER_ABBR)"${C_END}" > port:"${C_GRN}"$(DOCKER_PORT) is free to use."${C_END}; \
    else \
		echo ${C_BLU}"$(DOCKER_ABBR)"${C_END}" > port:"${C_RED}"$(DOCKER_PORT) is busy. Update ./.env file."${C_END}; \
	fi

# -------------------------------------------------------------------------------------------------
#  Enviroment
# -------------------------------------------------------------------------------------------------
.PHONY: env env-set env-laravel

LARAVEL_ENVIROMENT="./.env"
env-laravel: ## raplace any value of laravel .env file - $(s~^.*#)
	if [ -f ${LARAVEL_ENVIROMENT} ]; then \
        $(shell sed -i "s/DB_HOST=[^ ]*/DB_HOST=$(DOCKER_HOST)/g" ${LARAVEL_ENVIROMENT}) \
        $(shell sed -i "s/DB_PORT=[0-9\"]\+/DB_PORT=$(WEBADMIN_DB_PORT)/g" ${LARAVEL_ENVIROMENT}) \
        $(shell sed -i "s/DB_DATABASE=[A-Za-z0-9\"]\{1,\}/DB_DATABASE=$(WEBADMIN_DB_NAME)/g" ${LARAVEL_ENVIROMENT}) \
        $(shell sed -i "s/DB_USERNAME=[A-Za-z0-9\"]\{1,\}/DB_USERNAME=$(WEBADMIN_DB_USER)/g" ${LARAVEL_ENVIROMENT}) \
        $(shell sed -i "s/DB_PASSWORD=[^ ]*/DB_PASSWORD=\"$(WEBADMIN_DB_PASS)\"/g" ${LARAVEL_ENVIROMENT}) \
		echo ${C_GRN}"Laravel enviroment file has been updated"${C_END}; \
		echo ${C_YEL}"Checkout if it is correct before continue"${C_END}; \
    else \
		echo ${C_RED}"Laravel enviroment cannot be found"${C_END}; \
	fi

env: ## checks if docker .env file exists
	if [ -f ./docker/.env ]; then \
		echo ${C_BLU}"$(DOCKER_ABBR) DOCKER ENV"${C_END}" file "${C_GRN}"is set."${C_END}; \
    else \
		echo ${C_BLU}"$(DOCKER_ABBR) DOCKER ENV"${C_END}${C_YEL}" is not set."${C_END}" \
	Create it from root dir by "${C_YEL}"$$ make $(shell echo $(DOCKER_ABBR) | tr A-Z a-z)-env-set"${C_END}" \
	or from ./$(DIR_BASENAME)/ by "${C_YEL}"$$ make env-set"${C_END}; \
	fi

env-set: ## sets docker .env file
	echo "COMPOSE_PROJECT_NAME=$(DOCKER_NAME)\
	\nCOMPOSE_PROJECT_HOST=$(DOCKER_HOST)\
	\nCOMPOSE_PROJECT_PORT=$(DOCKER_PORT)" > ./docker/.env; \
	echo ${C_BLU}"$(DOCKER_ABBR) DOCKER ENV"${C_END}" file "${C_GRN}"has been set."${C_END}; \
    $(MAKE) env-laravel;

# -------------------------------------------------------------------------------------------------
#  Container
# -------------------------------------------------------------------------------------------------
.PHONY: ssh build install dev up start first stop restart clear

ssh:
	$(DOCKER_EXEC_TOOLS_APP)

build:
	cd docker && $(DOCKER_COMPOSE) up --build --no-recreate -d

install:
	cd docker && $(DOCKER_EXEC_TOOLS_APP) -c $(NODE_INSTALL)

dev:
	cd docker && $(DOCKER_EXEC_TOOLS_APP) -c $(SERVER_RUN)

up:
	cd docker && $(DOCKER_COMPOSE) up -d
	$(DOCKER_USER) docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(DOCKER_NAME)
	echo 127.0.0.1:$(DOCKER_PORT)

container-ip:
	$(DOCKER_USER) docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(DOCKER_NAME)
	echo $(DOCKER_PORT)

start:
	up dev

first:
	build install dev

stop:
	cd docker && $(DOCKER_COMPOSE) kill || true
	cd docker && $(DOCKER_COMPOSE) rm --force || true

restart:
	stop start dev

clear:
	cd docker && $(DOCKER_COMPOSE) down -v --remove-orphans || true
