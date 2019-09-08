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

Run `/bin/setup.sh` and follow the directions it prints at the end.

Update the following properties in `src/main/resources/.env.development`:

```
PHOENIX_BUILDER_URL=<your local phoenix builder url>
PHOENIX_BUILDER_API_BASE_URL=http://localhost:4400
```

In general, you'll need to keep your `.env.local` and/or `.env.development` files up to date with the values in `.env.sample`

### Running the Application

```bash
$ ./bin/web.sh
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

#### Mocking Data and Services in Tests
Below is a summary of how data and services are mocked in a testing environment.

*`GlobalSetup`* script empties the orchestration database and repopulates it with fixtures at the beginning of every test suite run. The seeded data can be found in `src/test/helpers/seed-constants.js` and in the `src/test/resources/sample-data` directory. The test environment, by default, uses the same orchestration database as the development environment.

*Snapshots* ([handled by Jest](https://jestjs.io/docs/en/snapshot-testing)) are used to compare the result of a test against an expected result. Snapshots are especially useful for testing complex, deterministic payloads.

Some of our tests use snapshots in order to check results against automatically-generated fixtures. If a snapshot test fails, then you may need to update them:

```bash
$ ./bin/update-snapshots.sh
```

And then we can use the `docker-copy` script to copy the newly generated snapshots from the Docker container to our host machine:

```bash
$ ./bin/docker-copy.sh -c phoenix-builder-service-_web_1 -s YOUR_PATH_TO_THE_CHANGED_TEST/__snapshots__
```

*Mocks* ([handled by Jest](https://jestjs.io/docs/en/manual-mocks)) are used to stub out functionality with mock data or a mock response. Services and middleware that touch remote resources are mocked in `__mock__` directories next to the real code and are explicity stubbed at the beginning of a test file with `jest.mock(/../../some/real/file/name.ts)`.

*  *Authentication*: Authentication includes conditional logic to simulate authorized and unauthorized users. To impersonate a user or access a route that requires auth, use the `createJWT` helper to create an auth token with a username, and email (optional), and attach it to the request.
  > Tip: when generating a valid token, usernames should be prefixed with `VALID`.

```ts
await request(app)
  .get("/some/route")
  .set("X-AUTH-TOKEN", createJWT("VALIDUSER1"));
```

#### Running Tests outside of Docker

1. Copy `cp ./src/main/resources/env/.env.test ./.env`
2. Run `./bin/test.sh` to start the database Docker
3. `yarn install`
4. `yarn jest --watch`

### Local Development

If you want to run a local [Phoenix Builder](https://github.com/daltino/phoenix-builder-angular) frontend against a local API service.

And run server:

```bash
$ bin/web-local
```

Note for `web-local`:
- API V3 relies on this Swagger server to authenticate.
- `web-local` must be run on both the frontend and backend.
- `web-local` must be run on frontend first.

See the Phoenix Builder [README](https://github.com/daltino/phoenix-builder-angular) for details.

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

We store some environment variables under `src/main/resources/env`. For environment variables that we need to keep
secret, we store those in AWS SSM, and load them just before starting the server (see code in the `entrypoint.sh` file)

Short descriptions of what each environment variable is for:

- `<ENV-VAR>` -  `<DESCRIPTION>`
