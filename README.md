# «YaMDb» project

![API for YaMDb project workflow](https://github.com/nekustetnaz/yamdb_final/actions/workflows/yamdb_workflow.yml/badge.svg?branch=master&event=push)

### Description
The YaMDb project collects user reviews on titles.
The titles are not stored in YaMDb, you cannot watch a movie or listen to music here.
The titles are divided into categories such as "Books", "Movies", "Music".
The list of categories can be expanded (for example, you can add the category "Fine Art" or "Jewelry").
A title can be assigned a genre from the preset list (for example, "Fairy Tale", "Rock" or "Arthouse").
Only an administrator can add titles, categories and genres.
Grateful or outraged users add reviews for the titles and give it a rating in the range from one to ten; an average rating of the title is formed from user ratings. 
User can add only one review for one title.
Users can comment on reviews.
Only authenticated users can add reviews, comments and ratings. 

### API YaMDb services
- Auth: authentication;
- Users: users;
- Titles: titles for reviews;
- Categories: categories of the titles. One title can only be linked to one category;
- Genres: genres of the titles. One title can be linked to several genres;
- Reviews: reviews of the titles. The review is linked to a specific title;
- Comments: comments on the reviews. The comment is linked to a specific review.

### User roles and access permissions
- Not authenticated user — can read descriptions of the titles, reviews, and comments;
- Authenticated user — can read everything, add reviews and rate titles, comment on reviews, edit and delete their reviews and comments, edit their ratings of titles. This role is assigned by default to each new user;
- Moderator — the same permissions as an Authenticated User. Additionally can delete and edit any reviews and comments;
- Admin — all permissions to manage all project content. Can create and delete titles, categories and genres. Can assign roles to users;
- Supepuser Django must always have administrator permissions. A superuser is always an administrator, but an administrator is not necessarily a superuser.

### Run project:
Automation of software deployment on servers is provided by the Docker virtualization environment as well as the Docker-compose tool.
Instructions for deploying the Docker container of a web application are provided in the Dockerfile:
```
FROM python:3.7-slim
RUN mkdir /app
COPY requirements.txt /app
RUN pip3 install -r /app/requirements.txt --no-cache-dir
COPY . /app
WORKDIR /app
CMD ["gunicorn", "api_yamdb.wsgi:application", "--bind", "0:8000"]
```

Instructions for launching all application containers are specified in the "docker-compose.yml" file:
```
version: '3.8'

services:
  db:
    image: postgres:13.0-alpine
    volumes:
      - /var/lib/postgresql/data/
    env_file:
      - ./.env
  web:
    build: nekustetnaz/api_yamdb
    restart: always
    volumes:
      - static_value:/app/static/
      - media_value:/app/media/
    depends_on:
      - db
    env_file:
      - ./.env
  nginx:
    image: nginx:1.21.3-alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
      - static_value:/var/html/static/
      - media_value:/var/html/media/
    depends_on:
      - web

volumes:
  static_value:
  media_value:
```

Template for filling variables in the .env file:
```
# Database type
DB_ENGINE=django.db.backends.postgresql

# Database type
DB_NAME=

# Login for connection to the database
POSTGRES_USER=

# Password for connection to the database
POSTGRES_PASSWORD=

# Database container name
DB_HOST=

# Database container port
DB_PORT=
```

To run the project in containers, type the command:
```
docker-compose up -d --build
```

After completing the build process and running the project, you should apply migrations, create a superuser, and collect static:
```
docker-compose exec web python manage.py migrate
docker-compose exec web python manage.py createsuperuser
docker-compose exec web python manage.py collectstatic --no-input
```

To fill in the database from a .csv file, type the command:
```
docker-compose exec web python manage.py closepoll
```

### Sign up a new user
- A user sends a POST request with an email and username parameters to the endpoint /api/v1/auth/signup/;
- The YaMDB service sends an email with a confirmation code to the specified email address;
- The user sends a POST request with the username and confirmation_code parameters to the endpoint /api/v1/auth/token/, in response to the request he receives a JWT token;
- As a result, the user receives a token and can use the project API by sending this token in each request;
- After signing up and receiving the token, the user can send a PATCH request to the endpoint /api/v1/users/me/ and fill in the fields in his profile.

### Technologies
Python 3 <br>
Django <br>
Django REST Framework <br>
Docker <br>
Gunicorn <br>
nginx <br>
PostgreSQL <br>
Simple JWT <br>

### Author
Anton Akulov - https://github.com/Nekustetnaz
