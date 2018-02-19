# go-project-guideline

Acoshift's Go Project Guideline

## OS

macOS ofc :D

Use macOS srsly, all script will write to use w/ macOS.

Another OS won't run the script and I don't want to maintain script for all OSes.

## Editors

- Recommend to use VSCode
- VIM/NVIM if you know what you're doing.

### VSCode Plugins

- Go
- ESLint
- markdownlint
- Sass
- Vetur
- Better TOML
- Beautify

### VSCode config

```json
{
    "editor.tabCompletion": true,
    "editor.snippetSuggestions": "inline",
    "files.trimTrailingWhitespace": true,
    "editor.insertSpaces": true,
    "editor.tabSize": 4,
    "editor.renderWhitespace": "all",
    "editor.rulers": [140],
    "go.liveErrors": {
        "enabled": true,
        "delay": 500
    },
    "files.exclude": {
        "**/.git": true,
        "**/.svn": true,
        "**/.hg": true,
        "**/CVS": true,
        "**/.DS_Store": true,
        "**/vendor/**": true,
        "**/node_modules/**": true
    },
    "[go]": {
        "editor.insertSpaces": false,
        "editor.formatOnSave": true,
        "editor.tabSize": 4
    },
    "[makefile]": {
        "editor.insertSpaces": false,
        "editor.tabSize": 4
    },
    "[javascript]": {
        "editor.insertSpaces": true,
        "editor.tabSize": 2
    },
    "[html]": {
        "editor.insertSpaces": true,
        "editor.tabSize": 2
    },
    "[scss]": {
        "editor.insertSpaces": true,
        "editor.tabSize": 2
    },
    "files.associations": {
        "*.tmpl": "html"
    }
}
```

## Softwares

```sh
# Go
brew install go
# or brew install go --devel

# dep
brew install dep

# gin live reload
go get -u github.com/codegangsta/gin

# Redis
brew install redis
brew services start redis

# PostgreSQL
brew install postgres
brew services start postgres

# NodeJS
brew install n
n lts

# SCSS
npm install -g node-sass

# jq
brew install jq

# postgres client
brew install pgcli

# postgres gui
open https://eggerapps.at/postico/

# HTTP client
open https://paw.cloud/

# below is for devops

# docker
open https://store.docker.com/editions/community/docker-ce-desktop-mac

# gcloud sdk https://cloud.google.com/sdk/
brew install python
cd ~
curl -O https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-sdk-189.0.0-darwin-x86_64.tar.gz
tar zxf google-cloud-sdk-189.0.0-darwin-x86_64.tar.gz
rm google-cloud-sdk-189.0.0-darwin-x86_64.tar.gz
./google-cloud-sdk/install.sh
source ~/.bash_profile
gcloud init
gcloud components install docker-credential-gcr gsutil beta

# kubectl
brew install kubectl
```

and run `brew update && brew upgrade` when you can.

## Project Structure

```
.
|-- Dockerfile
|-- Gopkg.lock
|-- Gopkg.toml
|-- Makefile
|-- README.md
|-- app
|-- assets
|   |-- favicon.png
|-- config
|-- deploy.yaml
|-- entity
|-- main.go
|-- migrate
|-- node_modules
|-- package.json
|-- repository
|-- styles
|   |-- config.scss
|   |-- main.scss
|   `-- utils.scss
|-- table.sql
|-- template
|   |-- _components
|   |-- _layout
|   |-- admin
|   |-- app
|   |-- auth
|   `-- main.tmpl
|-- vendor
`-- yarn.lock
```

> Only 1 .go file at root folder name main.go

### main.go

- load all configs using configfile from config folder
- init all vars ex. sql.DB, redis.Pool, google client

### config

- contains all config
- each file store 1 config

### table.sql

- contains latest version of schema

### migrate

- all migrate script if table schema changes
- include the first version of table.sql

### template

- contains all Go template
- all layout start with `_`

### entity

- contains web's entities

### repository

- contains functions to fetch data from database
- 1 function can do only 1 thing

### app

- contains all handlers
