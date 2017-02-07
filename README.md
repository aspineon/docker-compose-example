# docker-compose-example

## A simple Python web application

tips: https://docs.docker.com/compose/gettingstarted/

1.Create a directory for the project

```linux
$ mkdir compose-docker-test
$ cd compose-docker-test
```

2.Create a file called `app.py` in your project directory :

```python
from flask import Flask
from redis import Redis

app = Flask(__name__)
redis = Redis(host='redis', port=6379)

@app.route('/')
def hello():
    count = redis.incr('hits')
    return 'Hello from Docker! I have been seen {} times.\n'.format(count)

if __name__ == '__main__':
    app.run(host="0.0.0.0", debug=True)
```

3.Create another file called `requirements.txt`:

```
flask
redis
```

4.Create a Dockerfile that builds a Docker image(镜像). The image contains all the dependencies the Python application requires.

```Dockerfile
FROM python:3.4-alpine
ADD . /code
WORKDIR /code
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

This tells Docker to:
- Build an image starting with the Python 3.4 image.
- Add the current directory `.` into the path `/code` in the image.
- Set the working directory to `/code`.
- Install the Python dependencies.
- Set the default command for the container to `python app.py`.

5. Define services in a Compose file. Create a file called `docker-compose.yml` in your project directory.

```
version: '2'
services:
  web:
    build: .
    ports:
     - "5000:5000"
    volumes:
     - .:/code
  redis:
    image: "redis:alpine"
```

This Compose file defines two services, `web` and `redis`:
- Uses an image that's built from the `Dockerfile` in the current directory.
- Forwards the exposed port 5000 on the container to port 5000 on the host machine.
- Mounts the project directory on the host to `/code` inside the container, allowing you to modify the code without having to rebuild the image.

6.Build and run your app with Compose

* From your project directory, start up your application.
```
$ docker-compose up
```
Compose pulls a Redis image, builds an image for your code, and start the services you defined.

* Enter http://0.0.0.0:5000/ in a browser to see the application running.
* Refresh the page. The number should increment.

7.Experiment with some other commands

If you want to run your services in the background, you can pass the `-d` flag (for “detached” mode) to `docker-compose up` and use `docker-compose ps` to see what is currently running:
```
$ docker-compose up -d
$ docker-compose ps
```

The docker-compose run command allows you to run one-off commands for your services. For example, to see what environment variables are available to the web service:
```
$ docker-compose run web env
```
stop your services
```
$ docker-compose stop
```
