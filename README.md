# go-project-guideline

Acoshift's Go Project Guideline
for High Productivity and Maintainable w/o Test

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
# git
brew install git

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
|-- service
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

- Load all configs using configfile from config folder
- Init all vars ex. sql.DB, redis.Pool, google client

```go
func main() {
    rand.Seed(time.Now().UnixNano())

    config := configfile.NewReader("config")

    db, err := sql.Open("postgres", config.String("db"))
    if err != nil {
        log.Fatal(err)
    }

    app := app.New(&app.Config{
        DB:            db,
        SessionKey:    config.Bytes("session_key"),
        SessionSecret: config.Bytes("session_secret"),
        SessionStore: redisStore.New(redisStore.Config{
            Pool: &redis.Pool{
                Dial: func() (redis.Conn, error) {
                    return redis.Dial("tcp", config.String("session_host"))
                },
            },
            Prefix: config.String("session_prefix"),
        }),
        EmailDialer: gomail.NewPlainDialer(
            config.String("email_server"),
            config.Int("email_port"),
            config.String("email_user"),
            config.String("email_password"),
        ),
        EmailFrom:       config.String("email_from"),
        ReCaptchaSite:   config.String("recaptcha_site"),
        ReCaptchaSecret: config.String("recaptcha_secret"),
    })

    err = hime.New().
        TemplateDir("template").
        TemplateRoot("root").
        Minify().
        Handler(app).
        GracefulShutdown().
        ListenAndServe(":8080")
    if err != nil {
        log.Fatal(err)
    }
}
```

### config

- Contains all configs
- Each file store only 1 config (one line)

### table.sql

- Contains the latest version of database schema

### migrate

- Write migrate script if table schema changes
- Include the first version of table.sql
- Run though migrate will equals to run table.sql

### template

- Contains all Go template
- All layout *MUST* start with `_`

### entity

- Contains web's entities

### repository

- Contains functions to fetch data from database
- One function can do only one thing (ex. call db 1 time)
- All functions *MUST* *BE* stateless

### app

- Contains all handlers, template funcs
- Use global var for store config here

### service

- Contains complex logics (call repository multiple time in tx)

> always use pgsql.RunInTx if write to database multiple time
>
> DO NOT do long operation inside tx ex. verify password

### vendor

- Always push vendor to git
- And use prune config below

```toml
[prune]
  non-go = true
  go-tests = true
  unused-packages = true
```

## Workflow

- Everyone work on master
- Only push working code, DO NOT push anything broke, other peep can not work
- Always use git pull -r when can not push
- For new feature that will break current feature, use another branch
- Write good code at the start, we don't review your code
- Trust teammate code

## Testing

- DO NOT write test, you have 4294967295-1 things to do
- No one going to maintain your test, include yourself
- Test won't guarantee that it won't break, your code is
- Requirement will 100% change anyway

## Deployment

- Check any database schema changes, if it break wait until 2AM and scale down deployment to 0
- Build inside our computer
- Push to gcr.io
- Set new image to k8s
- Always use commit sha to tag docker

### k8s checklist

- Create new ingress (l7 load balancer)
- Fix domain for l7 ingress
- Use isolate secret to store tls in nginx-ingress
- Create configmap
- Set resource requests
- Set revision history to 3
- Set replicas to number of node -1
- Create deployment
- Create pod distrubtion budget

## Go Guideline

### Imports

Order imports by

- Native
- Lib
- Project

```go
import (
    "database/sql"
    "encoding/base64"
    "html/template"
    "io/ioutil"
    "math/rand"
    "net/http"
    "time"

    "github.com/acoshift/header"
    "github.com/acoshift/hime"
    "github.com/acoshift/httprouter"
    "github.com/acoshift/middleware"
    "github.com/acoshift/pgsql"
    "github.com/acoshift/session"
    "github.com/acoshift/webstatic"
    "github.com/dustin/go-humanize"
    "github.com/shopspring/decimal"
    "github.com/skip2/go-qrcode"
    "gopkg.in/gomail.v2"
    "gopkg.in/yaml.v2"

    "github.com/acoshift/project/entity"
    "github.com/acoshift/project/repository"
)
```

### Comment

- Write comment for all *export*

```go
// Config is the app's config
type Config struct {
    DB              *sql.DB
    SessionStore    session.Store
    SessionKey      []byte
    SessionSecret   []byte
    EmailDialer     *gomail.Dialer
    EmailFrom       string
    ReCaptchaSite   string
    ReCaptchaSecret string
}
```

```go
// FindAdminUsers finds user for admin view
func FindAdminUsers(q Queryer, username string) ([]*entity.AdminUser, error) {
    ...
}
```

- Write comments for complex logic *before* write code

```go
// Transfer transfer money from src user to dst user
func Transfer(db *sql.DB, srcUserID, dstUserID string, currency string, amount decimal.Decimal) error {
    return pgsql.RunInTx(db, nil func(tx *sql.Tx) error {
        // get src user's balance

        // verify src balance with amount

        // withdraw balance from src user

        // deposit balance to dst user

        return nil
    })
}
```

### Entity

- Split entity for each use case

```go
// User is the current user
type User struct {
    ...
}

// AdminUser is the user for admin view
type AdminUser struct {
    ...
}
```

### Repository

- DO NOT use `ID`, use userID, txID, etc.

- For struct, always returns pointer to struct `*entity.Type`

- For slice, always returns slice of pointer to struct `[]*entity.Type`

- For non-nil error, pointer *MUST* *NOT* nil, include slice (use zero length slice)

- Use plural for slice

- Always use `int64` for limit, offset, count

- `Get` for get data from ID

```go
func GetUser(q Queryer, userID string) (*entity.User, error) {
    ...
}

func GetUsername(q Queryer, userID string) (username string, err error) {
    ...
}
```

- `Find` for find from filters

```go
func FindPendingWithdrawTxs(q Queryer, limit, offset int64) ([]*entity.WithdrawTx, error) {
    ...
}
```

- `List` for list entities without filter

```go
func ListUsers(q Queryer, limit, offset int64) ([]*entity.User, error) {
    ...
}
```

- `Get` for aggregate data

```go
func GetUserCount(q Queryer) (cnt int64, err error) {
    ...
}

func GetUserBalance(q Queryer, userID string) (balance decimal.Decimal, err error) {
    ...
}

func GetUserTotalDeposit(q Queryer, userID string, currency string) (amount decimal.Decimal, err error) {
    ...
}
```

- `Set` for set data

```go
func SetUserPassword(q Queryer, userID string, hashedPassword string) error {
    ...
}
```

- `Remove` to delete data

```go
func RemoveToken(q Queryer, tokenID string) error {
    ...
}

func RemoveUserTokens(q Queryer, userID string) error {
    ...
}

func RemoveTokensCreatedBefore(q Queryer, before time.Time) error {
    ...
}
```
