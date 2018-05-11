# AntiPattern

Acoshift's Go Project Guideline
for High Productivity and Maintainable w/o Test

> DO NOT use this guideline if you don't know what you're doing :P

## OS

macOS ofc :D

Use macOS srsly, all script will write to use w/ macOS.

Another OS won't run the script and no one want to maintain script for all OSes.

## Editors

- Recommend to use VSCode
- VIM/NVIM if you know what you're doing.

### VSCode Plugins and Config

[vscode.md](https://github.com/acoshift/go-project-guideline/blob/master/vscode.md)

## Softwares

```sh
# git
brew install git

# Go
brew install go
# or brew install go --devel

# dep
brew install dep

# live reload
go get -u github.com/acoshift/goreload

# Redis
brew install redis
brew services start redis

# NodeJS
brew install n
n lts

# Nodejs dependencies
yarn global add node-sass gulp

# --- backend ---

# PostgreSQL (optional, normally use on cloud)
brew install postgres
brew services start postgres

# PostgreSQL Client
brew install pgcli

# PostgreSQL GUI
open https://eggerapps.at/postico/

# HTTP client
open https://paw.cloud/

# jq (optional)
brew install jq

# --- devops ---

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

```bash
$ tree .
.
|-- .gitignore
|-- Dockerfile
|-- Gopkg.lock
|-- Gopkg.toml
|-- Makefile
|-- README.md
|-- app
|-- assets
|   `-- favicon.png
|-- config
|-- deploy.yaml
|-- entity
|-- main.go
|-- migrate
|-- node_modules
|-- package.json
|-- repository
|-- service
|-- style
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
|-- Gulpfile.js
`-- yarn.lock
```

> Only 1 .go file at root folder name main.go

### Gulpfile.js

```js
const gulp = require('gulp')
const concat = require('gulp-concat')
const sass = require('gulp-sass')

const sassOption = {
  outputStyle: 'compressed',
  includePaths: 'node_modules'
}

gulp.task('default', ['style'])

gulp.task('style', () => gulp
  .src('./style/main.scss')
  .pipe(sass(sassOption).on('error', sass.logError))
  .pipe(concat('style.css'))
  .pipe(gulp.dest('./assets'))
)
```

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

ex. `config/session_host`
`localhost:6379`

config directory **MUST** equal to k8s ConfigMap

#### for configmap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: project
  labels:
    app: project
data:
  session_host: redis-1:6379
  session_prefix: 'project:'
```

### table.sql

- Contains the latest version of database schema

### migrate

- Write migrate script if table schema changes
- Include the first version of table.sql
- Run though migrate will equals to run table.sql

### template

- Contains all Go template
- All layout **MUST** start with `_`

### entity

- Contains entities for all use-cases

### repository

- Contains functions to fetch data from database
- One function can do only one thing (ex. call db 1 time)
- All functions **MUST** **BE** stateless

### app

- Contains all handlers, template funcs
- Use global vars here

### service

- Contains complex logics (call repository multiple time in tx)

> always use pgsql.RunInTx if write to database multiple time
>
> DO NOT do long operation inside tx ex. verify password

- All functions in service **MUST** **BE** stateless

### vendor

- Always push vendor to git
- And use prune config below

```toml
[prune]
  non-go = true
  go-tests = true
  unused-packages = true
```

### assets

- Try not to store anything in assets

- Use Google Storage to store assets, and set max-age to maximum with hash filename

## Workflow

- Everyone work on master (YES!!! on MASTER)
- Only push working code, DO NOT push any broken code, other peep can not work
- Always use `git pull -r` when can not (or before) push
- For new feature that will break current feature, use another branch
- Write good code at the start, no one will review your code
- Trust teammate code

## Testing

- DO NOT write test, you have 4294967295-1 things to do
- No one going to maintain your test, include yourself
- Test won't guarantee that it won't break, your code is
- Requirement will 100% change anyway

## Deployment

- Check any database schema changes, if it break wait until 2 AM and scale down deployment to 0
- Build inside our computer
- Push to gcr.io
- Set new image to k8s
- Always use commit sha to tag docker image

### k8s checklist

- Create new ingress (l7 load balancer)
- Fix domain for l7 ingress
- Use isolate secret to store tls in nginx-ingress
- Setup ingress's rate limit annotation for your use-case
- Create configmap
- Set resource requests
- Set revision history to 3
- Set replicas to number of node - 1
- Create deployment
- Create pod disruption budget

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

- Write comment for all **export**

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
// ListAdminUsers lists all users for admin view
func ListAdminUsers(q Queryer, limit, offset int64) ([]*entity.AdminUser, error) {
    ...
}
```

- Write comments for complex logic **before** write code

```go
// Transfer transfers money from src user to dst user
func Transfer(db *sql.DB, srcUserID, dstUserID string, currency string, amount decimal.Decimal) error {
    return pgsql.RunInTx(db, nil, func(tx *sql.Tx) error {
        // get src user's balance

        // verify src balance with amount

        // withdraw balance from src user

        // deposit balance to dst user

        return nil
    })
}
```

### Entity

- Split entity for each use-case

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

- DO NOT use `id`, use userID, txID, etc.

- For struct, always returns pointer to struct `*entity.Type`

- For slice, always returns slice of pointer to struct `[]*entity.Type`

- For non-nil error, pointer **MUST** **NOT** nil, include slice (use zero length slice)

- Use plural for slice, ex. Users, Tokens

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

- `Create` for insert data

### Repository function templates

- Select multiple rows

```go
func ListUsers(q Queryer, limit, offset int64) ([]*entity.User, error) {
    // always use lower case SQL
    rows, err := q.Query(`
        select
            id, username, email,
            created_at, updated_at
        from users
        order by created_at desc
        offset $1 limit $2
    `, offset, limit)
    if err != nil {
        return nil, err
    }
    defer rows.Close()

    xs := make([]*entity.User, 0)
    for rows.Next() {
        var x entity.User
        err = rows.Scan(
            &x.ID, &x.Username, &x.Email, // use line-break like SQL above
            &x.CreatedAt, &x.UpdatedAt,
        )
        if err != nil {
            return nil, err
        }
        xs = append(xs, &x)
    }
    if err = rows.Err(); err != nil {
        return nil, err
    }
    return xs, nil
}
```

- Select one row

```go
func GetUser(q Queryer, userID string) (*entity.User, error) {
    var x entity.User
    err := q.Query(`
        select
            id, username, email,
            created_at, updated_at
        from users
        where id = $1
    `, userID).Scan( // use line-break like SQL above
        &x.ID, &x.Username, &x.Email,
        &x.CreatedAt, &x.UpdatedAt,
    )
    if err != nil {
        return nil, err
    }
    return &x, nil
}
```

- Select w/ join (always use `left join` if possible), for `on` use joined table on the left

```go
func ListAdminTxs(q Queryer, limit, offset int64) ([]*entity.AdminTx, error) {
    rows, err := q.Query(`
        select
            t.id, u.username, t.currency, t.amount,
            t.type, t.created_at
        from transactions as t
            left join users as u on u.id = t.user_id
        order by t.created_at desc
        offset $1 limit $2
    `, offset, limit)
    if err != nil {
        return nil, err
    }
    defer rows.Close()

    xs := make([]*entity.AdminTx, 0)
    for rows.Next() {
        var x entity.AdminTx
        err = rows.Scan(
            &x.ID, &x.Username, &x.Currency, &x.Amount,
            &x.Type, &x.CreatedAt,
        )
        if err != nil {
            return nil, err
        }
        xs = append(xs, &x)
    }
    if err = rows.Err(); err != nil {
        return nil, err
    }
    return xs, nil
}
```

- Insert, create model if params too long

```go
// CreateUserModel is the model for CreateUser
type CreateUserModel struct {
    Username       string
    Email          string
    HashedPassword string
    ReferrerID     string
}

// CreateUser creates new user
func CreateUser(q Queryer, x *CreateUserModel) (userID string, err error) {
    err = q.QueryRow(`
        insert into users
            (
                username, email, password,
                referrer
            )
        values
            ($1, $2, $3, $4)
        returning id
    `,
        x.Username, x.Email, x.HashedPassword,
        sql.NullString{String: x.ReferrerID, Valid: x.ReferrerID != ""},
    ).Scan(&userID)
    return
}
```

- Update, use where for first placeholder

```go
func SetUserPassword(q Queryer, userID string, hashedPassword string) error {
    _, err := q.Exec(`
        update users
        set
            password = $2
            updated_at = now()
        where id = $1
    `, userID, hashedPassword)
    return err
}
```

### Handler

- Panic when unexpected error, that **CAN** **NOT** continue (fatal error for handler scoped)

```go
func must(err error) {
    if err != nil {
        panic(err)
    }
}

func profileGetHandler(ctx hime.Context) hime.Result {
    user := getUser(ctx)

    config, err := repository.GetUserConfig(db, user.ID)
    must(err)

    ...
}
```

- Use [hime](https://github.com/acoshift/hime) for handler standard

- Always have health check at `:18080/{readiness,liveness}`

```go
probe := probehandler.New()
health := http.NewServeMux()
health.Handle("/readiness", probe)
health.Handle("/liveness", probehandler.Success())

err = hime.New().
    // ...
    GracefulShutdown().
    Notify(probe.Fail).
    Wait(5 * time.Second).
    Before(func() { go http.ListenAndServe(":18080", health) }).
    ListenAndServe(":8080")
```

### Security Middlewares

- Reject CORS

```go
func noCORS(h http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        if r.Method == http.MethodOptions {
            http.Error(w, "Forbidden", http.StatusForbidden)
            return
        }
        h.ServeHTTP(w, r)
    })
}
```

- Add security headers [securityheaders.io](https://securityheaders.io/)

```go
func securityHeaders(h http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        w.Header().Set(header.XFrameOptions, "deny")
        w.Header().Set(header.XXSSProtection, "1; mode=block")
        w.Header().Set(header.XContentTypeOptions, "nosniff")
        h.ServeHTTP(w, r)
    })
}
```

- Filter **unused** methods

```go
func methodFilter(h http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        switch r.Method {
        case http.MethodGet, http.MethodHead, http.MethodPost:
            h.ServeHTTP(w, r)
        default:
            http.Error(w, "Method Not Allowed", http.StatusMethodNotAllowed)
        }
    })
}
```

- Protect CSRF w/ origin and referer (disable when development) [acoshift/csrf](https://github.com/acoshift/csrf)

```go
csrf.New(csrf.Config{
    Origins: []string{"https://yourwebsite"},
}),
```

## Style

- Add hash to file name

- Use Google Storage to store all style versions

- Always set max-age to maximum (31536000)

- Always use `-Z` when copy using gsutil, if directly use from Google Storage

> See Makefile section for more detail

## Makefile

```makefile
PROJECT=YOUR GCP PROJECT
IMAGE=YOUR DOCKER IMAGE REPOSITORY
ENTRYPOINT=YOUR COMPILED FILE NAME
BUCKET=YOUR BUCKET NAME
DEPLOYMENT=YOUR K8S DEPLOYMENT NAME
CONTAINER=YOUR K8S CONTAINER NAME

COMMIT_SHA=$(shell git rev-parse HEAD)
GO=go

dev:
    goreload -x vendor --all

clean:
    rm -f $(ENTRYPOINT)

style:
    gulp style

deploy-style:
    yarn install
    mkdir -p .build
    node-sass --output-style compressed --include-path node_modules style/main.scss > .build/style.css
    $(eval style := style.$(shell cat .build/style.css | md5).css)
    mv .build/style.css .build/$(style)
    gsutil -h "Cache-Control:public, max-age=31536000" cp -Z .build/$(style) gs://$(BUCKET)/$(style)
    echo "style.css: https://storage.googleapis.com/$(BUCKET)/$(style)" > .build/static.yaml

build: clean
    env GOOS=linux GOARCH=amd64 CGO_ENABLED=0 $(GO) build -o $(ENTRYPOINT) -ldflags '-w -s' main.go

docker: build deploy-style
    docker build -t $(IMAGE):$(COMMIT_SHA) .
    docker push $(IMAGE):$(COMMIT_SHA)

patch:
    kubectl set image deploy/$(DEPLOYMENT) $(CONTAINER)=$(IMAGE):$(COMMIT_SHA)

deploy: docker patch

rollback:
    kubectl rollout undo deploy/$(DEPLOYMENT)
```

## .gitignore

```.gitignore
.DS_Store
.goreload
private/
node_modules/
.build/
/assets/style.css
/projectname # compiled file name
```

## Dockerfile

```Dockerfile
FROM acoshift/go-scratch

ADD ENTRYPOINT / # your entrypoint
COPY template /template
COPY assets /assets
COPY .build/static.yaml /static.yaml
EXPOSE 8080

ENTRYPOINT ["/ENTRYPOINT"] # your entrypoint
```

## Finally

Programming, Motherfucker!
