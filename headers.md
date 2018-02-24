# HTTP Headers Checklist

- Vary: Accept-Encoding

Protect cache wrong encoding

- Cache-Control: no-cache, no-store, must-revalidate

For rendered HTML

- Cache-Control: public, max-age=31536000

For assets

- X-Content-Type-Options: nosniff

- X-Frame-Options: deny

- X-Xss-Protection: 1; mode=block

- Strict-Transport-Security: max-age=63072000; includeSubDomains; preload

- Content-Security-Policy

```
default-src 'self';
```

```
default-src 'none'; script-src 'self'; connect-src 'self'; img-src 'self'; style-src 'self';
```

- Referrer-Policy: same-origin

