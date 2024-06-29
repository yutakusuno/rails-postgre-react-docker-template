# rails-postgre-react-docker-template

This is a minimum template for setting up a project with Rails, PostgreSQL, React, and Docker in the development environment. It provides a quick start guide and detailed setup instructions.

## Quick start

Clone the repository:

```
git clone git@github.com:yutakusuno/rails-postgre-docker-template.git
cd rails-postgre-docker-template
```

Boot the application:

```
docker compose -f compose.dev.yaml up -d
```

Create the database:

```
docker compose -f compose.dev.yaml exec backend bundle exec rake db:create
```

After following these steps, you can access the frontend server at http://localhost:5173/ and the backend server at http://localhost:3000/.

Stop the application:

```
docker compose -f compose.dev.yaml down
```

## Setup from scratch

If you prefer to set up your project from scratch, follow these steps:

Create the project directories:

```
mkdir -p your-project-name/{frontend,backend}
cd your-project-name
git init
```

### Frontend

Set up the frontend:

```
cd frontend
pnpm create vite@latest .
```

Update `frontend/vite.config.ts`:

Add the frontend server host to run on Docker.

```diff
  export default defineConfig({
    plugins: [react()],
+   server: {
+     host: '0.0.0.0',
+   },
  });
```

Create a Dockerfile.dev:

```
touch Dockerfile.dev
```

Update frontend/Dockerfile.dev:

```dockerfile
# syntax=docker/dockerfile:1
FROM node:22-alpine

RUN npm install -g pnpm

WORKDIR /app
COPY package*.json ./
COPY . .

RUN pnpm install

EXPOSE 5173
CMD ["pnpm", "run", "dev"]
```

### Backend

Set up the backend:

```
cd backend
touch Dockerfile.dev
```

Update backend/Dockerfile.dev:

```dockerfile
# syntax = docker/dockerfile:1

# Make sure RUBY_VERSION matches the Ruby version in .ruby-version and Gemfile
ARG RUBY_VERSION=3.3.3
FROM registry.docker.com/library/ruby:$RUBY_VERSION-slim

# Rails app lives here
WORKDIR /app

# Install packages needed to build gems
RUN apt-get update -qq && \
  apt-get install --no-install-recommends -y build-essential git libpq-dev libvips pkg-config

# Install application gems
COPY Gemfile Gemfile.lock ./
RUN bundle install

# Copy application code
COPY . .

# Start the server by default, this can be overwritten at runtime
EXPOSE 3000
CMD ["./bin/rails", "server"]
```

Create a Gemfile and Gemfile.lock:

```
touch Gemfile Gemfile.lock
```

Update the Gemfile:

```
source 'https://rubygems.org'
gem 'rails', '~>7'
```

Leave Gemfile.lock empty. Then, create a `compose.dev.yaml` file at the project root directory:

```
cd ..
touch compose.dev.yaml
```

then, update the `compose.dev.yaml`

```yaml
services:
  db:
    image: postgres:16.3-alpine
    volumes:
      - ./backend/tmp/db:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
  backend:
    build:
      context: ./backend
      dockerfile: ./Dockerfile.dev
    command: bash -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0'"
    volumes:
      - ./backend:/app
    ports:
      - '3000:3000'
    depends_on:
      - db
  frontend:
    build:
      context: ./frontend
      dockerfile: ./Dockerfile.dev
    volumes:
      - ./frontend:/app
    ports:
      - '5173:5173'
```

Install frontend dependencies:

```
docker compose -f compose.dev.yaml run --rm frontend pnpm i
```

Create a new Rails application with an api mode:

```
docker compose -f compose.dev.yaml run --rm --no-deps backend rails new . --api --force --database=postgresql
```

Build the Docker images:

```
docker compose -f compose.dev.yaml build
```

Update `config/database.yml`:

Specify the db container.

```diff
  default: &default
    adapter: postgresql
    encoding: unicode
+   host: db
+   username: postgres
+   password: password
    pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>

  development:
    <<: *default
    database: app_development

  test:
    <<: *default
    database: app_test
```

Boot the app:

```
docker compose -f compose.dev.yaml up -d
```

Create the database:

```
docker compose -f compose.dev.yaml exec backend bundle exec rake db:create
```

After following these steps, you can access the frontend server at http://localhost:5173/ and the backend server at http://localhost:3000/.

Stop the application:

```
docker compose -f compose.dev.yaml down
```

## TDR

### An error caused by any of the subdirectory already having `.git`

At the project root directory:

```
> git add .

error: 'backend/' does not have a commit checked out
fatal: adding files failed
```

You can resolve the issue by deleting it:

**Please make sure to specify the right directory**.

```
rm -rf ./backend/.git/
```

Then, try again:

```
git add .
```
