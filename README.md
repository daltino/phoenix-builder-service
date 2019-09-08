# Phoenix Builder Service

This is the NodeJS backend service for Phoenix Builder. It exposes a series of API end-points that can be consumed to get words, english words, and word-groups for the various language suites available. It gives acess to the database and contains all the migrations.

## Key Architecture Components

- **Postgres** - Database
- **Express** - Webserver
- **AJV** - Schema validationve
- **Winston** - Logging
- **Knex** - Database migration and connectivity.
- **Jest** - Test suite
- **Docker** - Containerization

---

## Development

### Initial Setup

Run `/bin/init.sh` and follow the directions it prints at the end.

In general, you'll need to keep your `.env.local` and/or `.env.development` files up to date with the values in `.env.sample`

### Running the Application

```bash
$ ./bin/dev.sh
```

This will start the API server as well as the Postgres database. You can now interact with either of them from your host machine.

```bash
$ curl localhost:4400/health
{"status":"OK"}
```

```bash
$ psql pheonixDatabase nodejs -h localhost
phoenixDatabase=#
```

### Running the Test Suite

```bash
$ bin/test.sh
```

To run a specific test file or focus a test within a spec, add a few letters of the test name after the shell script; the script will match and run any test files.

#### Running Tests outside of Docker

1. Copy `cp ./src/main/resources/env/.env.test ./.env`
2. Run `./bin/test.sh` to start the database Docker
3. `yarn install`
4. `yarn jest --watch`

### Local Development

If you want to run a local [Phoenix Builder](https://github.com/daltino/phoenix-builder-angular) frontend against a local API service.

And run server:

```bash
$ bin/dev-local
```

### Git Strategy

We follow [Gitflow](https://nvie.com/posts/a-successful-git-branching-model/).  


When naming a branch, include the ticket number in the name (e.g. `PH-14`) so that someone reading the git history can easily navigate to the related ticket.  

Use "folders" (as singular nouns) to prefix branch-names, so that it is easy to categorize branches. Acceptable folders are `feature`, `spike`, `chore`, `hotfix`, and `release-candidate`.

An example branch name with all of these conventions is: `feature/PB-14-store-in-english-dict`.

When creating a PR, include the ticket number (e.g. `PB-14`) so that Bitbucket automatically creats a link over to the relevant JIRA ticket.

Ensure each commit is atomic and meaningful. Every commit should be a working state of the code and the commit history should read like a story. Squash commits that don't follow this pattern.

Use rebasing, amending, and other operations that modify history with caution.

## Staging

Bitbucket CI is set up to continuously deploy the `development` branch to AWS Staging.

The [health-check on AWS Staging](https://api.amabibi.com/v1/words/health) can be used to check the health status, DB status, and currently-deployed Git SHA.

## Production

Bitbucket CI is set up to continuously deploy the `master` branch to AWS Production.

[Here](https://api.amabibi.com/v1/words/health) is the health-check endpoint.

For production deploys:

1. Checkout development: `git checkout development`
2. Make sure you're up to date with origin: `git pull`
3. TBD

## Environment variables

We store some environment variables under `src/main/resources/env`.

Short descriptions of what each environment variable is for:

- `<ENV-VAR>` -  `<DESCRIPTION>`
