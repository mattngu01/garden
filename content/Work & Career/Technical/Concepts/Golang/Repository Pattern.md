https://gosamples.dev/postgresql-intro/repository-pattern/
https://deviq.com/design-patterns/repository-pattern

- decouple database & application logic
- "only one place to access domain obj data through data access layer"
	- "database implementation should be hidden behind an abstraction"

```go
type ProductRepository interface {
    Create(Product) error
    Get(productID uint) (*Product, error)
    Delete(productID uint) error
    List(limit, offset uint) ([]Product, error)
}
```

- used in Domain Driven Design (DDD)


program example:

```go
package repository

import (
    "context"
    "database/sql"
)

type UserRepository interface {
    Create(ctx context.Context, user *User) error
    GetById(ctx context.Context, id string) (*User, error)
    Update(ctx context.Context, user *User) error
    Delete(ctx context.Context, id string) error
}

type TrackRepository interface {
    Create(ctx context.Context, track *Track) error
    GetById(ctx context.Context, id string) (*Track, error)
    Update(ctx context.Context, track *Track) error
    Delete(ctx context.Context, id string) error
}

type Repository interface {
    UserRepository
    TrackRepository
}

type User struct {
    ID   string
    Name string
}

type Track struct {
    ID             string
    Name           string
    Danceability   float64
    Energy         float64
    MusicalKey     int8
    Loudness       float64
    Mode           float64
    Speechiness    float64
    Acousticness   float64
    Instrumentalness float64
    Liveness       float64
    Valence        float64
    Tempo          float64
}

func NewRepository(db *sql.DB) Repository {
    return &repository{db}
}

type repository struct {
    db *sql.DB
}

```
