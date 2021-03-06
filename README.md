# CPSC 449 Project\* 1

## Team Members:

Luc Dang

## Project Description:

- This project we created 2 RESTful microservices to mimic microblogging similar to Twitter
  - This project uses multiple python libraries such as Hug, sqlite_utils, and Requests to create
    2 APIs for both users and timelines
  - Users and posts are stored in separate databases, to authenticate user actions with their credentials
  - News users, posts, and followers can be created with the correct authentication

## Contents:

- `README.md`: README file for the project.
- `api_user.py`: API for the user microservice that handles various requsts for users such as retriveiving user data or their followers.
- `api_timeline.py`: API for the timeline microserve that handles the requests for timelines such as authentication for user's home timeline or retrieving a certain post.
- `.env`: environment file that sets the necessary environment variables for the project
- `Procfile`: procfile used by foreman to start the microservices

###

- `/bin/init.sh`: shell script to inialize the user and posts databases.
- `/bin/add_follower.sh`: shell script to add a follower to user `Arkadbe`. Can be edited as desired.
- `/bin/add_post.sh`: shell script to add a post from user `Arkadbe` with correct authentication. Can be edited as desired.
- `/bin/add_user.sh`:shell script to add a user `newUser` to the database. Can be edited as desired.

###

- `/share/...`: .csv files containing the template database created by me.

###

- `/var/placeholder.txt`: file that keeps the file in git commit. Ignore

## Instructions

- Extract the .tar file into a directory.
- Navigate into that directory and open a terminal or command prompt.
- Install necessary packages:

```
sudo apt update
sudo apt install --yes python3-pip ruby-foreman httpie sqlite3
python3 -m pip install hug sqlite-utils
sudo apt install --yes haproxy gunicorn
```

- Give all the bash scripts permission to execute with `chmod +x <SCRIPTNAME>.sh`
- Execute the init script with `./bin/init.sh`, which will intialize the databases
- Edit the HAProxy config file with `sudo nano /etc/haproxy/haproxy.cfg`
  appending the following:

```
frontend http_front
   bind *:80
   stats uri /haproxy?stats
   acl url_blog path_beg /users
   use_backend blog_back if url_blog
   default_backend http_back

backend http_back
   balance roundrobin
   server timeline1 127.0.0.1:8100 check
   server timeline2 127.0.0.1:8101 check
   server timeline3 127.0.0.1:8102 check

backend blog_back
   server userApi 127.0.0.1:8000 check
```

- Then restart HAProxy with `sudo systemctl restart haproxy`
- Start multiple instances of the microservices with forman using `forman start -m "api_user=1,api_timeline=3"`
- Open your browser and navigate to:
  - `localhost:8000` for the user microservice
  - `localhost:8000, :8001, :8002` for the timeline microservice
  - For API Documentation, see below
- Open a separate terminal window/tab to use HTTPie to interact with the APIs as well
  - Example `http -f GET :8100/posts/1`
- Examine the foreman terminal window to verify that timeline API requests are routed by HAProxy to different instances

## API Documentaion

### User API

| Actions                            | Description                                         |
| ---------------------------------- | --------------------------------------------------- |
| `GET /users`                       | Gets the list of users                              |
| `GET /users/{username}`            | Gets the user at the specified `username`           |
| `POST /users`                      | Creates a new user, if they don't already exist     |
| `GET /users/{username}/followers`  | Gets the followers list of the specified `username` |
| `POST /users/{username}/followers` | Creates a new follower for the specified `username` |

### Timeline API

| Actions                    | Description                                                                          |
| -------------------------- | ------------------------------------------------------------------------------------ |
| `GET /`                    | Gets the public timeline                                                             |
| `GET /timeline/{username}` | Gets all the posts of the specified `username`                                       |
| `GET /home`                | Using the authenticated user credentials, Gets all the posts of the user's followers |
| `GET /posts/{postId}`      | Gets the post specified by `postId`                                                  |
| `POST /posts`              | Using the authenticated user credentials, Creates a new post                         |
