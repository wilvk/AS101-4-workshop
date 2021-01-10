# Exercise 4 - Handling secrets

## TODO

- add ansible info to the setup instructions along with the installation guide (global vs virtual env)
- update readme, make it more readable and complete the TBD items 

## Prerequisites

If you haven't already done so, you will need to follow the [Setup Instructions](00_setup.md) before
continuing

## Remove secrets in code

We will update our code to remove secrets in code and instead, use a secret manager to source them.

1. Make sure you have ansible installed, see [Setup Instructions](00_setup.md) for details.
In case you went with virtual env option, first set up the shell properly (to make sure ansible commands are recognisable).
	
  ```
  pyenv local ansible-env
  ```

2. Encrypting sensitive variables

Create an encrypted file with credentials for postgres in it like so:
	`ansible-vault create env_secrets`
	You'd be asked to create a password for ansible vault. Make sure it's a strong password and store it somewhere secure, e.g. using a password manager.
  
  Once the password is entered, you'll need to paste the content of what you want to encrypt in the file in the following format.
  
  ```
  POSTGRES_USER="user"
  POSTGRES_PASSWORD="password"
  POSTGRES_DB="db-name"
  ```
  
Notes: Explain here why we don't need to add these variables to .env file - these would be picked up from the shell. TBD

3. Viewing and editing the encrypted file

In case you've made a mistake or you want to check what is currently stored in your encrypted file, use the following commands.
If you need to edit this file:
	
  `ansible-vault edit env_secrets`
  
Note that for edit command you can specify the preferred file editor you want to use. By default vi would be used. TBD - to add the instructions how this can be done.
To view the un-encrypted file contents:
	
  `ansible-vault view env_secrets`
  

For both of these commands you'd be asked to enter a password. For ease of use you could store the ansible-vault password locally, just make sure this is not in any of the repo directories. Then you can pass the password like so:

`ansible-vault view --vault-password-file=<path_to_vault_password> env_secrets`


4. Reference the encrypted variables in the docker-compose.yml file like so:
	
```
POSTGRES_USER: ${POSTGRES_USER}
POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
POSTGRES_DB: ${POSTGRES_DB}
```   
    
5. Running locally
   Export the previously encrypted variables to the shell environment:
	
  ```
  set +x
  export $(ansible-vault view --vault-password-file=<path_to_vault_password> env_secrets|xargs)
  ```
  
Note: `set +x` makes sure we are not logging out the value of the variables to the standard output.

To test what your docker-compose file would look like after the substitution run:

	`docker-compose config`

Rebuild the containers from scratch and test your system works as expected (there should not be any errors and you should be able to add new)
	
```
docker-compose rm -f
docker-compose pull  
docker-compose up --build -d
```

Once test passed, you can stop the containers:
	`docker-compose stop -t 1`

Commit and push your changes. Note, you'd need to update the .talismanrc file in order to commit your changes.

6. Running in a pipeline

Notice that one of your workflows just failed - lint_test. The reason for this is that you've extracted the postgres variables, but haven't specified where these values should be taken from. When we run our application locally, we've been exposing the variable values through the shell. To do similar things when running github actions, we'd need to do some updates.

First of all, you'd need to provide a source of the vault password.
Luckily, github offers an option to add the secrets via: 
Your forked repo -> Settings -> Secrets -> 'New repository secret'

TBD <insert the screenshot of the secret here>

Then you'd need to make sure that lint_test workflow has access to the variables encrypted in env_secrets file.
TODO :explain the usage of action.yml, Dockerfile, entrypoint.sh and how the lint_test.yml needs to be updated.

Note: we need the following lines to make sure the content of the postgres variables is not printed in the job logs.
```
- name: hide credentials from the output
      run: |
        echo "::add-mask::$POSTGRES_USER"
        echo "::add-mask::$POSTGRES_PASSWORD"
        echo "::add-mask::$POSTGRES_DB"
```  

There's a known issue for this, where the variables get logged in plain text the 1st time after they've been masked
Issue link - https://github.com/actions/runner/issues/475


Note that for entrypoint.sh you have to have proper permissions: 
`chmod +x entrypoint.sh`

