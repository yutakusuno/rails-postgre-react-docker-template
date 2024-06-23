# rails-postgre-react-docker-template

This is a minimum template for setting up a project with Rails, PostgreSQL, React, and Docker in the development environment. It provides a quick start guide and detailed setup instructions.

## Quick start

Clone the repository:

```
git clone git@github.com:yutakusuno/rails-postgre-docker-template.git
cd rails-postgre-docker-template
```

Start the Docker containers:

```
docker compose up -d
```

Create the database:

```
docker-compose run --rm backend bundle exec rake db:create
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
npm create vite@latest .
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

Create a Dockerfile:

```
touch Dockerfile
```

Update frontend/Dockerfile:

```dockerfile
# syntax=docker/dockerfile:1
FROM node:22-alpine3.19
WORKDIR /app
COPY package*.json ./
COPY . .
RUN npm install
EXPOSE 5173
CMD ["npm", "run", "dev"]
```

### Backend

Set up the backend:

```
cd backend
touch Dockerfile
```

Update backend/Dockerfile:

```dockerfile
# syntax=docker/dockerfile:1
FROM ruby:3.3.2
RUN apt-get update -qq && apt-get install -y postgresql-client
WORKDIR /app
COPY Gemfile Gemfile.lock ./
COPY . .
RUN bundle install
EXPOSE 3000
CMD ["rails", "server", "-b", "0.0.0.0"]
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

Leave Gemfile.lock empty. Then, create a `compose.yaml` file at the project root directory:

```
cd ../
touch compose.yaml
```

then, update the `compose.yaml`

```yaml
services:
  db:
    image: postgres:16.3-alpine3.20
    volumes:
      - ./backend/tmp/db:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
  backend:
    build: ./backend
    command: bash -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0'"
    volumes:
      - ./backend:/app
    ports:
      - '3000:3000'
    depends_on:
      - db
  frontend:
    build: ./frontend
    volumes:
      - ./frontend:/app
    ports:
      - '5173:5173'
    depends_on:
      - backend
```

Install frontend dependencies:

```
docker compose run --rm frontend npm i
```

Create a new Rails application with an api mode:

```
docker compose run --rm --no-deps backend rails new . --api --force --database=postgresql
```

Update backend/Dockerfile:

```diff dockerfile
+ ENV RAILS_ENV="development" \
- ENV RAILS_ENV="production" \
    BUNDLE_DEPLOYMENT="1" \
    BUNDLE_PATH="/usr/local/bundle" \
-   BUNDLE_WITHOUT="development"
+   BUNDLE_WITHOUT="production"
```

Build the Docker images:

```
docker compose build
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
docker compose up -d
```

Create the database:

```
docker-compose run --rm backend bundle exec rake db:create
```

After following these steps, you can access the frontend server at http://localhost:5173/ and the backend server at http://localhost:3000/.

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
