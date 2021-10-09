# Create a Discord slash command with a FastAPI server running on SAP BTP

## Docs

  - Python Discord Bot Library: 
    - Extended library providing __Slash Commands__: `discord-py-interactions`
      - [GitHub](https://github.com/goverfl0w/discord-interactions)
      - [Docs](https://discord-interactions.readthedocs.io/en/latest/)
    - Core library that is used by `discord-py-interactions`: `discord.py`
      - [GitHub](https://github.com/Rapptz/discord.py)
      - [Docs](https://discordpy.readthedocs.io/en/latest/)

  - Discord Slash Commands reference: 
    https://discord.com/developers/docs/interactions/slash-commands

  - Superhero database API: https://akabab.github.io/superhero-api/api/


## Authorize Discord application

  1. Create a Discord server (called a _guild_ in Discord wording).
  1. Create a new Discord application under https://discord.com/developers.
  1. Add a _bot_ to the application.
  1. Go to section _OAuth2_: 
     1. Select _SCOPES_:
        - `applications.commands`
        - `bot`
          - `Use Slash Commands`
          - `Send Messages`
     1. Copy the URL and open it in the browser to authorize the application to be used 
        in the Discord server/guild.
  1. Take the bot token: _Bot | Click to Reveal Token_.


## Create the Python app

  1. `pipenv install discord discord-py-slash-command`
  1. `pipenv install python-dotenv`
  1. `pipenv install requests`
  1. Create `app/main.py`.


## Local development

  Launch the app:
  `python app/bot.py --env-file .env.dev`


## Deploy to BTP

  1. Derive the `requirements.txt` from the `Pipfile`: 
     `pipenv lock --keep-outdated -r > requirements.txt`

  1. Create a `manifest.yml` that contains the deployment configuration:
     ```yaml
      ---
      applications:
      - name: discord-slash-cmd
        buildpacks:
          - python_buildpack
        host: xi3k
        path: .
        memory: 128M
        env:
          DEV: 'True'
        command: python app/main.py
        health-check-type: none
     ```
  1. Create a `runtime.txt` containing the Python version that is required to run the 
     application. Available versions are listed in the CF buildpack release 
     [notes](https://github.com/cloudfoundry/python-buildpack/releases).
     ```
     python-3.9.x
     ```

  1. Provide the dependencies as "vendor dependencies" such that no network calls are 
     required during build time:
     `pip download -d vendor -r requirements.txt --platform manylinux1_x86_64 --platform manylinux2014_x86_64 --only-binary=:all:`

  1. Login to the CF environment on the BTP account:
     1. Prerequisites:
        1. SAP BTP account with subaccount exists.
        1. Cloud Foundry space is available in the subaccount.
     1. To get the _API endpoint_ required for login:
        <a id="cf_get_api_endpoint"></a>
        1. Select the subaccount.
        1. Go to _Overview_.
        1. On tab _Cloud Foundry Environment_ see _API Endpoint_.
     1. Run `cf login`, provide the API endpoint and the credentials used to login into the 
        BTP account.

  1. Push the application:
     `cf push`
     __Note__: The following warnings can be __ignored__:
     - _WARNING:  The script %s is installed in '%s' which is not on PATH._
     - _No start command specified by buildpack or via Procfile._
     - _App will not start unless a command is provided at runtime._


## Business Application Studio (BAS)

  1. Create an OAuth access token for the Git remote repo.
  1. In BAS, clone the repo and provide the access token.
  1. Install Python by following this 
     [guide](https://blogs.sap.com/2020/12/12/xtending-business-application-studio-3-of-3/).
  1. Install extension _Python_.
  1. `pip`should be available on `PATH`. Install `pipenv`:
     `pip install pipenv`
  1. Start the virtual environment and install the project dependencies:
     - `pipenv shell`
     - `pipenv install`
  1. Install the extension _Python_. __Ignore__ the error that there is no Python 
     installed.
  1. Set the Python interpreter such that the extension works:
     - _File | Settings | Open Preferences... | User | Python | Default Python Interpreter_
     - Set `/home/user/<Python version>/bin/python`
     - Do the same in the _Preferences_ under _Workspace_.
     - To set the Python environment for the concrete project folder, 
       create/edit file `.vsocde/settings.json` and add an entry 
       `"python.pythonPath": "/home/user/.local/share/virtualenvs/<virtual env>/bin/python"`

  1. If _Linter pylint is not installed._ pops up, select _Install_. This will install it
     using `pip`.

  1. Create a _run and debug configuration_:
     1. _Run | Add Configuration..._
     1. Provide the following contentn to `launch.json`:
        ```json
          "version": "0.2.0",
          "configurations": [
              {
                "name": "Python: Current File",
                "type": "python",
                "request": "launch",
                "program": "${file}",
                "console": "integratedTerminal"
              },
              {
                "name": "Python: Discord Bot",
                "type": "python",
                "request": "launch",
                "program": "${workspaceFolder}/app/main.py",
                "console": "integratedTerminal",
                "envFile": "${workspaceFolder}/.env.dev"
              }      
            ]
        }
        ```

