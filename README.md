# Gigalixir Action

This action will deploy your elixir application to Gigalixir and will run your migrations automatically.

Note: This action has only been tested in one repo and has no unit tests.

## Usage

```yaml
test: 
  # A job to run your tests, linters, etc

deploy:
  needs: test # Will only run if the test job succeeds
  if: github.ref == 'refs/heads/main' # Only run this job if it is on the main branch

  runs-on: ubuntu-latest

  steps:
    - uses: actions/checkout@v2
      with:
        ref: main # Check out main instead of the latest commit
        fetch-depth: 0 # Checkout the whole branch
        
    - uses: actions/setup-python@v2
      with:
        python-version: 3.8.1
        
    - uses: mhanberg/gigalixir-action@<current release>
      with:
        APP_SUBFOLDER: my-app-subfolder  # Add only if you want to deploy an app that is not at the root of your repository
        GIGALIXIR_APP: my-gigalixir-app # Feel free to also put this in your secrets
        GIGALIXIR_CLEAN: true # defaults to false
        GIGALIXIR_USERNAME: ${{ secrets.GIGALIXIR_USERNAME }}
        GIGALIXIR_PASSWORD: ${{ secrets.GIGALIXIR_PASSWORD }} # Use GIGALIXIR_PASSWORD or GIGALIXIR_API_KEY
        GIGALIXIR_API_KEY: ${{ secrets.GIGALIXIR_API_KEY }} # Use GIGALIXIR_PASSWORD or GIGALIXIR_API_KEY
        MIGRATIONS: false  # defaults to true
        SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
```
## Gigalixir CLI Authentication (MFA modifications from closed pull request of nickdichev (https://github.com/gigalixir/gigalixir-action/pull/48))
The default strategy for authenticating the Gigalixir CLI logs in with `GIGALIXIR_USERNAME` and `GIGALIXIR_PASSWORD`. However, if MFA is enabled on the account the login prompt becomes interactive (asking for the MFA token).

However, the authentication actually shoves an API key into your `~/.netrc` which the [CLI uses](https://gigalixir.readthedocs.io/en/latest/cli.html?highlight=netrc#authentication) when executing commands. You can find your API key on the Gigalixir dashboard under Account > API Key.

By supplying `GIGALIXIR_API_KEY` instead of `GIGALIXIR_PASSWORD` you can enable MFA on the account associated with deploying the application and still use this action! See the [Gigalixir documentation](https://gigalixir.readthedocs.io/en/latest/account.html?highlight=api%20key#how-to-use-multi-factor-authentication) for the steps to enable MFA.

## Migrations

Currently running migrations is only supported when your app is deployed as a mix release.

The migrations are run with the `gigalixir ps:migrate` command, which requires having a public key uploaded to your app's container and a private key locally to connect via an `ssh` connection.

Please see the docs for [How to Run Migrations](https://gigalixir.readthedocs.io/en/latest/main.html#migrations) for more information.

If your migrations fail, the action will rollback the app to the last version.

## Contributing

Remember to 

- `npm install`
- `npm run package`
