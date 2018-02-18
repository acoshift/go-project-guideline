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
; Go
brew install go
; or brew install go --devel

; dep
brew install dep

; gin live reload
go get -u github.com/codegangsta/gin

; Redis
brew install redis
brew services start redis

; PostgreSQL
brew install postgres
brew services start postgres

; NodeJS
brew install n
n lts

; SCSS
npm install -g node-sass

; jq
brew install jq

; below is for devops

; docker
open https://store.docker.com/editions/community/docker-ce-desktop-mac

; gcloud sdk https://cloud.google.com/sdk/
brew install python
cd ~
curl -O https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-sdk-189.0.0-darwin-x86_64.tar.gz
tar zxf google-cloud-sdk-189.0.0-darwin-x86_64.tar.gz
rm google-cloud-sdk-189.0.0-darwin-x86_64.tar.gz
./google-cloud-sdk/install.sh
source ~/.bash_profile
gcloud init
gcloud components install docker-credential-gcr gsutil beta

; kubectl
brew install kubectl
```

and run `brew update && brew upgrade` when you can.
