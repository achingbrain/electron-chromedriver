# docker-electron-chromedriver

Runs an instance of [electron-chromedriver](https://www.npmjs.com/package/electron-chromedriver) as a Docker container and includes a VNC server to watch the tests run.

In a nutshell you build your app, mount the build folder as a volume inside this Docker container using `docker-compose` then configure an instance of [Selenium Webdriver](https://www.npmjs.com/package/selenium-webdriver) to connect to chromedriver inside the container and run your tests.

Currently only works on Linux hosts as it requires [fuse](https://github.com/libfuse/libfuse) inside the container.

## How to use

Create a `docker-compose.yml` file similar to the following:

```yml
chromedriver:
  image: paulwib/docker-electron-chromedriver:2.0.0
  volumes:
    - /dev/fuse:/dev/fuse
    - ./build:/app # Your app will be available in the /app dir in the container
  environment:
    SCREEN_WIDTH: 1920
    SCREEN_HEIGHT: 1080
    # Other configuration options here
  ports:
    - "9515:9515" # Chromedriver port, host:container
    - "5900:5900" # VNC server port, host:container
  cap_add:
    - SYS_ADMIN
  privileged: true
```

Configure electron-chromedriver with the following env vars:

```
CHROMEDRIVER_PORT 9515
CHROMEDRIVER_WHITELISTED_IPS ""
CHROMEDRIVER_OPTS
SCREEN_WIDTH 1920
SCREEN_HEIGHT 1080
SCREEN_DEPTH 24
```

Finally create a driver and use it to run your tests:

```javascript
const webdriver = require('selenium-webdriver')

const driver = new webdriver.Builder()
  .usingServer('http://localhost:9515')
  .withCapabilities({
    chromeOptions: {
      binary: '/app/path-to-my-app'
    }
  })
  .forBrowser('electron')
  .build()
```

The port in the URL passed to `usingServer` should match the host port that the container's port `9515` is mapped to (`9515` by default) and the path to the binary should be changed to match where your app's binary is, e.g. `/app/My App-1.0.0-x86_64.AppImage` or similar.

## VNC Server

There is a built in VNC server running inside the container on port `5900` by default.  To connect, use the password `secret`.

E.g:

```shell
$ vncviewer localhost::5900
# when prompted, enter the password 'secret'
```
