# SQL Guideline

1. Always use lower case.
1. Indent with tab
1. Always use not null, until you need null value
1. Always set default to datatype default value (ex. int = 0, varchar = '', bool = false), except timestamp can set to now()

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
		email, password, name, is_admin, is_ban,
		created_at
	)
values
	(
		$1, $2 $3, $4, $5, $6
	)
returning id;
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
	password = excluded.password
returning id;
```

### Select

```sql
select
	email, password, is_admin,
	created_at, updated_at
from users
where email = $1;
```

```sql
select
	email, password, is_admin,
	created_at, updated_at
from users
where
		is_admin = true
	and	is_ban = false
order by created_at desc
limit 100
offset 200;
```

```sql
select
	u.username, t.value, t.created_at
from t1 as t
	left join users as u on u.id = t.user_id
order by created_at desc;
```

```sql
select
	u.username, t.value, t.created_at
from t1 as t
	left join users as u
		on
				u.id = t.user_id
			and	t.created_at > now() - '7d'
order by created_at desc;
```

```sql
select
	date at time zone 'UTC+7' at time zone 'UTC',
	sum(case when type = 't1' and c = 'A' then -a else 0 end) as a,
	sum(case when type = 't2' and c = 'B' then a else 0 end) as b,
	sum(case when type = 't2' and c = 'C' then a else 0 end) as c
from
	(
		select
			date_trunc($1, created_at at time zone 'UTC+7') as date,
			c, type,
			sum(a) as a
		from t7
		where
				(type = 't1' and c = 'A')
			or	(type = 't2' and c = 'B')
			or	(type = 't2' and c = 'C')
		group by
			date_trunc('day', created_at at time zone 'UTC+7'),
			c,
			type
	) as t
group by date
order by date desc
offset 60
limit 30;
```

### Delete

```sql
delete from users
where is_ban = true;
```
