**Install** 
`go get -u github.com/cosmtrek/air`

**Setup**
`air init`

**.air.toml (clean architecture)**
```yaml
root = "."
testdata_dir = "testdata"
tmp_dir = "tmp"

[build]
  args_bin = []
  bin = "./tmp/main"
  cmd = "go build -o ./tmp/main ./cmd/main.go"
  delay = 0
  exclude_dir = ["assets", "tmp", "vendor", "testdata"]
  exclude_file = []
  exclude_regex = ["_test.go"]
  exclude_unchanged = false
  follow_symlink = false
  full_bin = ""
  include_dir = []
  include_ext = ["go", "tpl", "tmpl", "html"]
  include_file = []
  kill_delay = "0s"
  log = "build-errors.log"
  poll = false
  poll_interval = 0
  rerun = false
  rerun_delay = 500
  send_interrupt = false
  stop_on_error = false

[color]
  app = ""
  build = "yellow"
  main = "magenta"
  runner = "green"
  watcher = "cyan"

[log]
  main_only = false
  time = false

[misc]
  clean_on_exit = false

[screen]
  clear_on_rebuild = false
  keep_scroll = true
```

**Dockerfile**
```dockerfile
FROM golang:1.22.1-alpine

ARG GITHUB_TOKEN
WORKDIR /app

RUN apk add git mercurial
RUN go install github.com/cosmtrek/air@latest

COPY go.mod go.sum ./

RUN apk add git mercurial
RUN git config --global url."https://${GITHUB_TOKEN}:x-oauth-basic@github.com/".insteadOf "https://github.com/"

RUN go mod download

CMD ["air", "-c", ".air.toml"]
```

**docker-compose.yaml**
```yaml
version: "3.8"

services:
  web:
    build:
      context: .
      dockerfile: Dockerfile
      # Correct the path to your Dockerfile
      args:
        - GITHUB_TOKEN=${GITHUB_TOKEN:-token}
    image: app
    container_name: app
    restart: unless-stopped
    env_file: .env
    ports:
      - "$PORT:$PORT"
    depends_on:
      - postgres
    volumes:
      - ./:/app

  postgres:
    image: postgres:14.0
    container_name: postgres
    restart: unless-stopped
    env_file: .env
    environment:
      - POSTGRES_USER=$DB_USER
      - POSTGRES_PASSWORD=$DB_PASS
      - POSTGRES_DB=$DB_NAME
    ports:
      - "$DB_PORT:$DB_PORT"
    volumes:
      - dbdata:/var/lib/postgresql/data

volumes:
  dbdata:
```

**How to run**
`docker compose up postgres -d`
`docker compose up web`