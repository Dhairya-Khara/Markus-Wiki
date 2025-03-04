# Developer Guide: Set Up with Docker

## Downloading and Installing

If you want to get started on working on MarkUs quickly and painlessly, this is the way to do it.

1. If you are using **Windows**, you will need to install Windows Subsystem for Linux (WSL) by following the instructions on [this page](https://docs.microsoft.com/en-us/windows/wsl/install-win10). (The "Simplified Installation" section is probably easiest, but you need to join the Windows Insiders Program with a Microsoft account.)

    - If you are given a choice of what operating system to use, select *Ubuntu 20.04*.

2. Install [Docker](https://docs.docker.com/get-docker/) and [Docker Compose](https://docs.docker.com/compose/install/).

    - On Windows, make sure you've selected the "WSL 2 backend" tab under "System Requirements" and follow those instructions.
    - On Linux, also follow the instructions on "Manage Docker as a non-root user" [here](https://docs.docker.com/install/linux/linux-postinstall/).

3. If you are using **Windows**, you'll need to open a terminal into the WSL system you installed. *This is the terminal where you'll type in the rest of the commands in this section.*

    1. To start the WSL terminal, open the start menu and type in "ubuntu". Click on the "Ubuntu 20.04" application. (We recommend pinning this to your taskbar to make it easier to find in the future.)
    2. Type in the command `pwd`, which shows what folder you're currently in. You should see `/home/<your user name>` printed. If it isn't, switch to your home directory using the command `cd ~`.

4. Clone the Markus repository from GitHub by following the instructions in [Setting up Git and MarkUs](Developer-Guide--Setting-up-Git.md). (This is a document you will want to read very carefully and may come back to.)

5. Change into the repository that you just cloned: `cd Markus`.

6. Run `docker-compose build app`.

7. Run `docker-compose up rails`. The first time you run this it will take a long time because it'll install all of MarkUs' dependencies, and then seed the MarkUs application with sample data before actually running the server. When the server actually starts, you'll see some terminal output that looks like:

    ```text
    Puma starting in cluster mode...
    * Version 4.3.1 (ruby 2.5.3-p105), codename: Mysterious Traveller
    * Min threads: 0, max threads: 16
    * Environment: development
    * Process workers: 3
    * Preloading application
    * Listening on tcp://0.0.0.0:3000
    Use Ctrl-C to stop
    - Worker 0 (pid: 185) booted, phase: 0
    - Worker 1 (pid: 193) booted, phase: 0
    - Worker 2 (pid: 201) booted, phase: 0
    ```

    1. On **Windows**, open a separate WSL terminal (but leave the current one running), and type in the command `docker-compose run --rm rails bash`. This will take you into the Docker container; you'll see the prompt change to `app/$`.

    2. Run the command `rails js:routes`, which will take a moment to generate a required file.

8. Open your web browser and type in the URL `localhost:3000/csc108`. The initial page load might be slow, but eventually you should see a login page. Use the username `instructor` and any non-empty password to login.

    *Tip*: to terminate the Rails server, go to the terminal window where the server is running and press `Ctrl + C`/`⌘ + C`.

    - On Windows Home Edition, you'll need to use the Docker container's IP address instead: `192.168.99.100:3000/csc108`.

9. In a new terminal window, go into the Markus directory again and run `docker-compose run rails rspec` to run the MarkUs test suite. This will take several minutes to run, but all tests should pass.

    **Troubleshooting**

    - If you see a test failing with the following message near the top:

        ```text
        1) SubmissionsController#get_file When the file is a jupyter notebook file should download the file as is
           Failure/Error: _stdout, stderr, status = Open3.capture3(*args, stdin_data: file_contents)

           Errno::ENOENT:
               No such file or directory - /app/nbconvertvenv/bin/jupyter-nbconvert
        ```

    Run the following commands:

    ```console
    docker-compose run --rm rails bash  # This takes you into the Docker container
    python3.8 -m venv /app/nbconvertvenv
    /app/nbconvertvenv/bin/pip install wheel nbconvert
    ```

    Then try re-running the tests. You can do this from your current terminal (inside the Docker container) simply by running `rspec`.

Hooray! You have MarkUs up and running. Please keep reading for our recommended developer setup.

## Installing and Configuring RubyMine

We strongly recommend RubyMine (a JetBrains IDE) for all MarkUs development.

1. First, install RubyMine from [here](https://www.jetbrains.com/ruby/download). Note that if you are a current university student, you can obtain a [free license](https://www.jetbrains.com/student/) for all JetBrains software.

2. Open the MarkUs repository in RubyMine.

    - On Windows, your repository will be located at `\\wsl$\Ubuntu-20.04\home\<your user name>\Markus`.

3. Complete the setup steps under [Docker: Enable Docker Support JetBrains guide](https://www.jetbrains.com/help/ruby/docker.html#enable_docker).

4. To configure RubyMine to use a remote Ruby interpreter from the Docker image: [JetBrains guide](https://www.jetbrains.com/help/ruby/using-docker-compose-as-a-remote-interpreter.html#set_compose_remote_interpreter). Use `rails` as the service. After you've selected this interpreter, RubyMine will take some time to index all of the Ruby gems (libraries); you'll see "Indexing"... at the bottom of the RubyMine window.

    If this doesn't work, please make sure you're using the latest version of RubyMine (Help -> Check for Updates...).

5. To configure RubyMine to connect to the PostgreSQL database that MarkUs uses for development, first make sure the MarkUs server is running (by doing a `docker-compose run...` as above). Then follow [these instructions](https://www.jetbrains.com/help/idea/running-a-dbms-image.html#6aa07130) in RubyMine to connect to the PostgreSQL server running as the 'postgres' docker-compose service. Note that you do not need to create a new container so you should only need to follow the instructions under "Connect to the PostgreSQL server".

    - hostname: `localhost`
    - port: `35432`
    - user: `postgres`
    - password: `docker`
    - database: `markus_development`

## Installing Pre-Commit Hooks

We use [pre-commit](https://pre-commit.com/) to run automated checks on code before each commit. To set this up on your local computer (not in Docker):

1. First, install Python 3.
2. Then, install the pre-commit library: `$ python3 -m pip install pre-commit` (or just `python` instead of `python3`, depending on your Python executable.
3. Finally, in the `Markus` folder run `$ pre-commit install`. This will install all of the Markus pre-commit hooks.

After this, these checks will run every time you make a commit. If all checks pass, the commit will proceed as normal. If a check fails, the commit *does not* occur, and there are two possibilities:

- Some checks will automatically fix issues (e.g., most style checks). *These changes still need to be manually git added and committed!*
- Some checks will just report problems that you'll need to fix manually. After fixing them, you'll need to add and commit those changes.

## Running Commands in Docker

Here's a summary of the few most common tasks you'll use in your development.

- Start the MarkUs server: `docker-compose up --no-recreate rails`
- Run the MarkUs rspec test suite: `docker-compose run rails rspec`
- Run a specific rspec test file: `docker-compose run rails rspec FILE`
- Run the Markus Jest test suite:  `docker-compose run rails yarn test`
- Run the Markus Jest test suite with the test coverage shown:  `docker-compose run rails yarn test-cov`
- Run a specific Jest test file: `docker-compose run rails yarn test FILE`
- Start a shell within the Docker Rails environment: `docker-compose run --rm rails bash`.
  Within this shell, you can:
    - Install new dependencies: `bundle install`, `yarn install`
    - Reset the MarkUs database: `rails db:reset`
    - Run a database migration: `rails db:migrate`
    - Start the interactive Rails console: `rails c`

Here's a summary of a few commands that are helpful for managing containers.

- Stop the MarkUs server: `docker-compose stop rails`
- Start the MarkUs server up again (after stopping it): `docker-compose start rails`
- Remove all containers started by MarkUs: `docker-compose down`
- Remove all containers and all volumes started by MarkUs: `docker-compose down -v`
    - Note that removing volumes will mean that you will lose all changes made in the database

If you need to rebuild the MarkUs docker image:

- Stop and remove the existing containers and remove all volumes: `docker-compose down -v`
- Do steps 5 and 6 from the Downloading and Installing section [above](#downloading-and-installing)

## Setting up the autotester

**Note**: you only need to consult this section if you'll be working with the MarkUs autotester.

1. Clone the [markus-autotesting repo](https://github.com/MarkUsProject/markus-autotesting). Don't clone it into your `Markus` folder; we recommend cloning it into the same parent folder as your `Markus` folder.
2. `cd` into the `markus-autotesting` folder.
3. Run `docker compose build` to build a new Docker images for the MarkUs autotester.
4. Run `docker compose up` to create the new containers. The first time you run this it will take a long time because it'll install all of the MarkUs autotester's dependencies.
    You'll know it's done when you see "INFO success..."
5. Stop the containers by pressing Ctrl + C (Windows/Linux) or Cmd + C (macOS). Then, restart the containers by running the command `docker compose start`.
6. In a separate terminal, start the MarkUs server: `docker compose up rails`.
7. In a web browser, visit the running server, but using a different domain than `localhost`:
    - For Windows and macOS, visit `host.docker.internal:3000/csc108`. If that doesn't work:
        - Windows: first open a WSL terminal and enter the command `ip addr show eth0 | grep inet`. Use the IP address found after `inet`, which is a sequence of 4 numbers separated by `.`, e.g. `100.20.200.2`. Try visiting `<IP address>:3000/csc108` instead.
        - For macOS, visit `docker.for.mac.localhost:3000/csc108` instead.
    - For Linux, visit `172.17.0.1:3000/csc108`.
8. Now, open a shell in the MarkUs docker container: `docker-compose run --rm rails bash`.
9. Execute the following commands in the MarkUs container.
    1. Create sample autotesting assignments: `rails db:autotest`.
    2. (*The MarkUs server and autotest containers be running when you run these commands.*) Run tests for every sample autotesting asignment: `MARKUS_URL=<URL> rails db:autotest_run`, where `<URL>` is in the form `http://<DOMAIN>:3000`, and `<DOMAIN>` is the domain you used in Step 7 (e.g., `host.docker.internal`).

        If you get an error when running this command, see "Running tests manually" below.

Now when you visit MarkUs in the web browser, you should see the new assignments that were created, the autotest settings (under Settings -> Automated Testing), and a sample submission with autotest results.

### Running tests manually

If the `rails db:autotest_run` fails, you can still run the tests manually in your web browser by doing the following:

1. Go to MarkUs in your web browser (using the same URL as Step 7 above).
2. Navigate to the `autotest_custom` assignment (under the Assignments tab), and go to Settings -> Automated Testing. This will take you to the settings page for the automated tests.
3. On that page, change the "Timeout" field from 30 to 60, and press "Save" at the bottom of the page. You should see a message at the top of the page that shows the status of updating the settings; wait until this message changes to "Completed".
4. Now go to the "Submissions" tab to view a table of all submissions---in this case, there will be just one. Click on the link in the leftmost column of the table. This takes you to the grading view for the submission.
5. Go to the Test Results tab and click on "Run Tests".
6. Wait a minute, and then refresh the page. Go back to the Test Results tab. You should see that two tests have been run, and that both have passed.
7. Repeat for the other assignments that you want to run tests for.

## Setting up ActionMailer

If you plan on doing work that involves sending/recieving emails from MarkUs, you will need to [configure ActionMailer](https://guides.rubyonrails.org/action_mailer_basics.html). To get you started quickly on setting up ActionMailer and understanding what it is used for in MarkUs, follow the instructions outlined in [Enabling ActionMailer In Development](Developer-Guide--Tips-And-Tricks--Enabling-ActionMailer-In-Development.md).

## Troubleshooting

**Note: This is an archive of problems related to Docker that are encountered by students, and their solutions.**

### Q

I'm writing frontend code. The files I've changed should according to the Webpack config files trigger Webpack rebuild, but that's not happening. I've verified that

1. My changes are valid and should be displayed from the URL I'm accessing.
2. There are no errors in the Webpacker container's logs.
3. If I run `yarn build-dev` in the Webpacker container's console directly, it succeeds and I'm able to see my changes afterwards.

### A

*This solution is experimental and could lead to problems such as higher CPU usage.* This is likely due to Webpack's `watch` option not working properly. According to the official Webpack [docs](https://webpack.js.org/configuration/watch/#watchoptionspoll), one suggestion when `watch` is not working in environments such as Docker, is to add `poll: true` to `watchOptions` inside the Webpack config file, which in our case, is `webpack.development.js`. This should help resolve the problem.
