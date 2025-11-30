---
title: Markdown page example
---

# Markdown page example

You don't need React to write simple standalone pages.

## Testing

How to test `auth/login`

```sh
curl -X POST http://localhost:3000/auth/login -d '{"username": "John", "password": "pass"}' -H "Content-Type: application/json"
```

How to test `profile`

```sh
curl http://localhost:3000/profile -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyTmFtZSI6IkpvaG4iLCJzdWIiOjEsImlhdCI6MTc2Mzg1MTE0OCwiZXhwIjoxNzYzODUxMjA4fQ.Z60oggzfHqLlazvv_BadkWT9v7JMtYn2g5pMp9On4J8"
```
