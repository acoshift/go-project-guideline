# SQL Guideline

1. Always use lower case.
1. Indent with tab
1. Always use not null, until you need null value
1. Always set default to datatype default value (ex. int = 0, varchar = '', bool = false)

## Example

### Create Table

```sql
create table users (
    id uuid default gen_random_uuid(),
    email varchar not null,
    password varchar not null,
    is_admin bool not null default false,
    created_at timestamp not null default now(),
    updated_at timestamp not null default now(),
    primary key (id)
);
create unique index on users (email);
create index on users (created_at desc);
```

### Insert

```sql
insert into users
    (email, password)
values
    ($1, $2);
```

```sql
insert into users
    (
        email, password,
        is_admin, created_at
    )
values
    (
        $1, $2
        $3, $4
    );
```

### Update

```sql
update users
set
    is_admin = $2,
    updated_at = now()
where id = $1;
```

### Upsert

```sql
insert into users
    (email, password)
values
    ($1, $2)
on conflict (email) do update
set
    password = excluded.password;
```

### Select

```sql
select
    email, password, is_admin,
    created_at, updated_at
from users
where email = $1
```

```sql
select
    email, password, is_admin,
    created_at, updated_at
from users
order by created_at desc
limit 100
offset 200;
```
