# Setting-up
*This does not covers installation of docker and compose*

*Also, presuming myapp is already created*

*For this example RoR app Will have*

- Web server
- Postgres DB

### Inside rails app create Dockerfile
We'll start with the Dockerfile, which describes how to build our virtual environment to host a Rails app. 

```sh
FROM ruby:2.2.0
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev
RUN mkdir /myapp
WORKDIR /myapp
ADD Gemfile /myapp/Gemfile
RUN bundle install
ADD . /myapp
```

### Inside rails app docker-compose.yml
Docker Compose lets us define our containers and how they should be linked together with a simple yaml file.
```sh
db:
  image: postgres
  ports:
    - "5432"
web:
  build: .
  command: bundle exec rails s -p 3000 -b '0.0.0.0'
  volumes:
    - .:/myapp
  ports:
    - "3000:3000"
  links:
    - db
```

### Inside rails app change database.yml

```sh
default: &default  
  adapter: postgresql
  encoding: unicode
  username: postgres
  host: <%= ENV['MYAPP_DB_1_PORT_5432_TCP_ADDR'] %>
  port: <%= ENV['MYAPP_DB_1_PORT_5432_TCP_PORT'] %>
  pool: 5

development:  
  <<: *default
  database: myapp_development

test:  
  <<: *default
  database: myapp_test
```
### We can run the following command to see all ENV Variables from our service
```sh
docker-compose run web env
```

### Inside rails app add the following line to GEMFILE
```sh
gem 'therubyracer',  platforms: :ruby
```

### Building Containers
```sh
docker-compose build
````

### Setting up database and migrations
```sh
docker-compose run web rake db:create
docker-compose run web rake db:migrate
```

### Running the app 

```sh
docker-compose up
````

www.gabriel.pw
