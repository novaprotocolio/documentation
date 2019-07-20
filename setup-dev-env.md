# How to setup env for development

## Step 1: prerequisite

Make sure you have:

- docker & docker-compose
- yarn
- go

## Step 2: basic services

start redis, local-ethereum-node and PostgresDB

```bash
docker-compose up redis ethereum-node db
```

## Step 3: nova services

start other Nova Backend Services

```bash
# change to sub-dir: backend
cd ./backend

# API
make api

# engine
make engine

# launcher
make launcher

# watcher
make watcher

# websocket
make ws

# go back to project-dir
cd ..
```

## Step 4: web

```bash
# change to sub-dir: web
cd ./web

# install dependencies
yarn install

# start web
yarn start

# go back to project dir
cd ..
```

## Step 5: run service

```bash
# start and watch api changes
yarn watch api
```
