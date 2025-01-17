wpt-pr-bot
======

A helper bot for web-platform-tests.

Authoring tests
=====

For the purposes of automated testing, this project uses [the `replay`
module](https://www.npmjs.com/package/replay) to record responses from the
GitHub HTTP API. Contributors seeking to add new tests or modify existing tests
will need to capture new request/response pairs. This can be achieved as
follows:

1. Create a "personal access token" though GitHub.com (no particular access
   scopes are necessary)
2. Set the `GITHUB_TOKEN` environment variable to the value created above, set
   the `REPLAY` environment variable to `record`, and run the tests.
3. If the tests pass, include the files that were generated by the previous
   command in the patch.

Development
=====

Requirements:
- Node v16

### Getting Started

Download the dependencies

```bash
npm install
```

### Running Tests

To lint and run the tests, run:

```bash
npm run test
```

Deploying
=====

1. Get the secrets:
   - ```bash
     npm run gcp-build
     ```
   - TODO: Can remove step 1 after #213
2. Deploy:
   - ```bash
     # Create a variable to represent the version
     VERSION_ID="rev-$(git rev-parse --short HEAD)"

     # Deploy the new version but do not promote it
     gcloud --project=wpt-pr-bot app deploy --no-promote -v "${VERSION_ID}"
     ```
3. Check the logs for the instance
   - ```bash
     gcloud --project=wpt-pr-bot app logs tail -s default -v "${VERSION_ID}"
     ```
   - Check to make sure that the app has started. It should
     [print](https://github.com/web-platform-tests/wpt-pr-bot/blob/main/index.js#L121)
     `App started in ....`
4. Redirect traffic to the new version.
   - ```bash
     gcloud --project=wpt-pr-bot app services set-traffic default \
         --splits="${VERSION_ID}"=1
     ```
5. Keep only 2 versions of the app (VERSION_ID and the previous version).
   To delete older versions:
   - ```bash
     # Check the number of versions. There should only be two.
     gcloud --project=wpt-pr-bot app versions list --service=default
     # If there are more than two. Get the version id.
     # Given old version ${REALLY_OLD_VERSION_ID}, delete it.
     gcloud --project=wpt-pr-bot app versions delete \
         ${REALLY_OLD_VERSION_ID} --service=default
     ```
6. Monitor the "Recent Deliveries"
[tab](https://github.com/web-platform-tests/wpt/settings/hooks/161784539?tab=deliveries)
in wpt. If there are any failures, use the command in step 3 to tail the logs.
