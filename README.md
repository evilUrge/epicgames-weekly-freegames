# Automatically get the epicgames store weekly free games
> A fork of `ricosorio/epicgames-weekly-freegames` with ARM support!

I like free games but I don't like repeating the same process over and over again when it can be automated... And that's why I made this!

## Important to read first
This was an introductory project to web scrapping for me and not only the script would stop working with any updates to the site html but since this project started EPIC implemented a mandatory 2FA to redeem the free weekly games... because of that I no longer maintain this proejct.
Any pull requests are welcome but I recommend you check out @charlocharlie's approach to this problem using their API instead! https://github.com/charlocharlie/epicgames-freegames-node

## What it is
This is a simple python3.7 script making use of [selenium webdriver](https://selenium.dev/) and [chrome driver](https://sites.google.com/a/chromium.org/chromedriver/) to run run chrome in headless mode, navigate to the epicgames store, login into your account and redeem all weekly free games available. All of this inside of a docker container.

## Getting started
Personally I avoid hosting such processes on my machine so I don't have to worry about having it turned ON during the times when it's supposed to run. So I opt for deploying it to AWS FARGATE and completely forget it exists... until I check my account on the website that is! ;)

#### Locally (Docker)
To run it locally, pull the image from docker-hub:
```
docker pull docker pull evilurge/epicgames-weekly-freegames:latest
```
And run a docker container from the newly downloaded image:
```
docker run -e EMAIL=<EMAIL> -e PASSWORD=<PASSWORD> evilurge/epicgames-weekly-freegames
```
Replacing the environment variables `EMAIL` and `PASSWORD` for your epicgames store credentials.

#### AWS Cloud (Fargate)
To run the image AWS Fargate (ECS) start by creating a task definition that that pulls the image from `pull evilurge/epicgames-weekly-freegames:latest`. Note that you don't need to specify the host (by default it pulls from docker-hub).

When it comes to how much resources the container needs, I find it enough to have 0.5GB and lowest version of CPU - 0.25.

## Optional configuration
Different options can be provided to alter the output or execution of the program. All available options are:

- `TIMEOUT` and `LOGIN_TIMEOUT` if you have a slow internet connection speed and want to make sure it won't affect the result of the script. Defaults to `TIMEOUT = 5` and `LOGIN_TIMEOUT = 10`.

- `LOGLEVEL` can be used to specify the [log level](https://docs.python.org/3.7/library/logging.html#logging-levels) to log. Defaults to `INFO`

- `SLEEPTIME` can be used to set the number of seconds to wait between looping through the procedure again. Defaults to `-1`, which will stop execution after a single iteration.

- `TOTP` is used to log in to an account protected with 2FA. If you have 2FA enabled already, you may have to redo it in order for to obtain your TOTP token. Go [here](https://www.epicgames.com/account/password) to enable 2FA. Click "enable authenticator app." In the section labeled "manual entry key," copy the key. Then use your authenticator app to add scan the QR code. Activate 2FA by completing the form and clicking activate. Once 2FA is enabled, use the key you copied as the value for the `TOTP` parameter.

- `DEBUG` allows the script to be run locally without having to manually edit the chromedriver path (instead will searches in the current dir) and will ignore the headless flag when starting up chrome. Only checks for the presence of this environment variable, its value is not interpreted.

Running a docker container with these:
```
docker run -e TIMEOUT=15 -e LOGIN_TIMEOUT=20 -e LOGLEVEL=DEBUG -e SLEEPTIME=43200 -e EMAIL=<EMAIL> -e PASSWORD=<PASSWORD> evilurge/epicgames-weekly-freegames
```

## Docker Compose
```
version: '2'

services:

    egs-freegame:
        build:
            context: ./epicgames-weekly-freegames/
            dockerfile: Dockerfile
        restart: always
        environment:
            - TIMEOUT=10
            - LOGIN_TIMEOUT=15
            - SLEEPTIME=43200
            - LOGLEVEL=DEBUG
            - EMAIL=example@example.com
            - PASSWORD=password123
```

## Login with Cookies
If the program closes because it encounters in a captcha you can try to login through cookies. To do this you must:
- Open a browser and enter to [epic games store](https://www.epicgames.com/store/en-US/)
- Login making sure that you have checked the "remember me" box
- Install an extension on your browser to export cookies. I have used [this](https://chrome.google.com/webstore/detail/editthiscookie/fngmhnnpilhplaeedifhccceomclgfbg) but also [this](https://chrome.google.com/webstore/detail/cookie-editor/hlkenndednhfkekhgcdicdfddnkalmdm) should work
- Open the extension and export all the cookies (fifth button starting from the left in the top bar of the extension that I used)
- Paste the cookies into the file called cookies.json
- Restart the script

>To copy a cookie file in a swarm\compose, bind the local copy of `cookies.json` to: `/tmp/cookies.json`