# Domain Driven Development (DDD)

Acoshift's Go Project Guideline for Low Productivity but Maintainable w/ Test

> Recommend for use this guileline for large project.
>
> DO NOT use in small project, you will end up write boilerplate code

## Project Structure

```
.
|-- .gitignore
|-- Dockerfile
|-- Makefile
|-- README.md
|-- domain1
|   |-- service
|   |   |-- subdomain1.go # service interface for subdomain1
|   |   |-- subdomain2.go # service interface for subdomain2
|   |-- subdomain1
|   |-- subdomain2
|   |-- transport1
|   |   |-- subdomain1
|   |   |   |-- endpoint.go
|   |   |   |-- transport.go
|   |   |-- subdomain2
|   |-- transport.go 
|-- domain2
|   |-- service.go # service interface and implement for domain2
|   |-- domain2.go # entities, and repository interface for domain2
|   |-- transport.go # mount service layer into transport layer
|   |-- endpoint.go # wrap service action with transport-specific handler
|-- domain3 # shared domain
|   |-- domain3.go # entities, and repository interface for domain3
|-- domain4
|-- storage1
|   |-- domain2.go # implement domain2 repository for storage1
|   |-- domain3.go # implement domain3 repository for storage1
|-- storage2
|   |-- domain4.go # implement domain4 repository for storage2
|-- httptransport
|   |-- httptransport.go # http transport layer shared functions
|   |-- middleware.go # http transport related middlewares
|-- websockettransport
|   |-- websockettransport.go # websocket transport layer shared functions
|   |-- middleware.go # websocket transport related middlewares
|-- grpctransport
|   |-- grpctransport.go # grpc transport layer shared functions
|   |-- middleware.go # grpc transport related middlewares
|-- api # shared global data type ex. errors
|   |-- errors.go # global errors
|   |-- db.go # db interface
|-- main.go
|-- table.sql
|-- migrate
|   |-- 0000-init-database.sql
|   |-- 0001-add-user-status.sql
|-- data # shared data
|   |-- thai_provinces.json
|-- internal # shared logic, general data types ex. custom datetime type
|   |-- internal1
|   |   |-- internal1.go
|-- config
|   |-- config1
|   |-- config2
|-- assets # for web assets
|   |-- landing # landing assets
|   |-- webadmin # webadmin assets
|-- landing # landing page
|   |-- service
|   |   |-- landing.go # landing service interface
|   |-- service.go # implement for landing service
|   |-- handler.go # http handler/controller factory
|   |-- index.go # index related handler
|   |-- faq.go # faq related handler
|-- webadmin # web admin
|-- template # html template
|   |-- landing # landing's templates
|   |   |-- main.tmpl
|   |   |-- _layout
|   |-- webadmin # web admin's templates
|-- mock # mock repositories
|   |-- user.go # mock user repository
```
## Domains

Domain is the thing that you interest.

For example

- `auth` is a domain for authentication. `auth` domain can **import** other **shared** **domain**
- `profile` is a domain for current user's profile.

```go
// profile/service.go

package profile

// Service is the profile service
type Service interface {
    // Get returns user's profile
    Get(userID string) (*Profile, error)

    // Update updates user's profile
    Update(profile *Profile) error

    // ChangePhoto changes user profile's photo
    ChangePhoto(userID string, r io.Reader) error
}
```

for upload user's photo, file repository is required to do this.

```go
// file/file.go

package file

// Repository is the file storage
typs Repository interface {
    // Store stores data in r and return its download url
    Store(r io.Reader, baseDir string) (downloadURL string, err error)
}
```

for get, update data for user, user repository is required

```go
// user/user.go

package user

// Repository is the user storage
type Repository interface {
    // CreateUser creates new user and return created id
    CreateUser(db api.DB, user *User) (userID string, err error)

    // GetUser gets user by id
    GetUser(db api.DB, userID string) (*User, error)

    // SetPhoto sets user's photo
    SetPhoto(db api.DB, userID string, photoURL string) error
}
```

then the `profile` service may require file, and user repository

```go
// profile/service.go (continue)

// NewService creates new profile service
func NewService(db api.DB, users user.Repository, files file.Repository) Service {
    return &service{db, users, files}
}

type service struct {
    db       api.DB
    users    user.Repository
    files    file.Repository
}
```

### Shared Domain

Shared Domain is the domain that shared with other domains.

Shared Domain **SHOULD** **NOT** import other domains to avoid cycle dependency.

In the example above, `user`, and `file` are shared domains.

## Transport

Transport is the layer between transport to service.

### Endpoint

Endpoint is the endpoint for each transport.

For Example

```go
// profile/service.go

package profile

// Service is the profile service
type Service interface {
    // Get returns user's profile
    Get(userID string) (*Profile, error)

    // Update updates user's profile
    Update(profile *Profile) error

    // ChangePhoto changes user profile's photo
    ChangePhoto(userID string, r io.Reader) error
}
```

We can create endpoint for profile service like this.

```go
// profile/httpendpoint.go

type getResponse struct {
    Profile
}

// makeGetEndpoint creates http's endpoint for Get
func makeGetEndpoint(s Service) http.Handler {
    return http.HandlerFunc(w http.ResponseWriter, r *http.Request) {
        userID := app.GetUserID(r.Context())
        resp, err := s.Get(userID)
        if err != nil {
            httptransport.HandleError(w, r, err)
            return
        }
        httptransport.HandleSuccess(w, r, &getResponse{*resp})
    })
}
```

then we can use this endpoint in transport like this.

```go
// profile/httptransport.go

// MakeHandler creates new profile http handler
func MakeHandler(s Service) http.Handler {
    m := http.NewServeMux()

    m.Handle("/profile", makeGetEndpoint(s))

    return httptransport.ChainMiddleware(
        httptransport.MustSignedIn(),
    )(mux)
}
```

## Controller

Controller uses to handle multiple-page application (MPA).

```go
// landing/handler.go

package landing

import (
    "path/to/profile/internal/filesystem"
    // other imports ...
)

// New creates new landing handler
func New(ctrl Controller) http.Handler {
    m := http.NewServeMux()
    m.Handle("/-/", http.StripPrefix("/-", filesystem.New("assets/landing")))

    r := httprouter.New()
    r.Get("/", ctrl.Index)
    r.Get("/faq", ctrl.Faq)

    m.Handle("/", r)

    return httptransport.ChainMiddleware(
        httptransport.RejectCORS(),
        httptransport.RequireSession(),
    )(m)
}
```

```go
// landing/landing.go

package landing

import (
    "path/to/project/api"
    "path/to/project/landing/service"
    "path/to/project/internal/renderer"
)

// Controller is the landing controller
type Controller interface {
    Index http.HandlerFunc
    Faq   http.HandlerFunc
}

// NewController creates new landing controller
func NewController(db api.DB, landings service.Landing) Controller {
    c := &ctrl{landings}
    // load templates to renderer
    return c
}

type ctrl struct {
    renderer renderer.Renderer
    landings service.Landing
}

func (c *ctrl) Index(w http.ResponseWriter, r http.Request) {
    c.renderer.View("index", nil)
}
```

## Testing

## Service

### Mock Repository

```go
// mock/user.go

// NewUserRepository creates new mock user repository
func NewUserRepository() user.Repository {
    //
}
```

### Testing

```go
// profile/service_test.go

package profile_test

func TestGetNotFound(t testing.T) {
    mockDB := mock.NewDB()
    mockUsers := mock.NewUserRepository()
    mockFiles = mock.NewFileRepository()
    s := profile.NewService(mockDB, mockUsers, mockFiles)
    resp, err := s.Get("not exists user id")
    if err != profile.ErrNotExists {
        t.Errorf("expected Get not found user returns ErrNotExists, got %v", err)
    }
    if resp != nil {
        t.Errorf("expected Get not found user returns nil response; got %v", resp)
    }
}
```

## Endpoint

```go
// profile/endpoint_internal_test.go

package endpoint

// mock service, use your own mocking style
typs mockService struct {
    GetFunc         func(userID string) (*Profile, error)
    ChangePhotoFunc func(userID string, r io.Reader) error
}

func (s *mockService) Get(userID string) (*Profile, error) {
    return s.GetFunc(userID)
}

func (s *mockService) ChangePhoto(userID string, r io.Reader) error {
    return s.ChangePhotoFunc(userID, r)
}

func TestGetHandler(t testing.T) {
    s := &mockService {
        GetFunc: func(userID string) (*Profile, error) {
            return nil, ErrNotFound
        },
    }
    h := makeGetEndpoint(s)
    req := httptest.NewRequest(...)
    resp := httptest.NewResponseRecoder(...)
    h.ServeHTTP(w, r)

    // check response
}
```

