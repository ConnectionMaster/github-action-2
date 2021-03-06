# 🚧 This is work in progress, see [#1](https://github.com/probot/github-actions/pull/1)

# `@probot/github-action` Adapter

> A `probot` drop-in replacement for running Probot apps in GitHub Actions

## Usage

Create your Probot Application as always

```js
// app.js
module.exports = (app) => {
  app.on("issues.opened", async (context) => {
    const params = context.issue({ body: "Hello World!" });
    await context.octokit.issues.createComment(params);
  });
};
```

Then in the entrypoint of your GitHub Action, require `@probot/github-action` instead of `probot`

```js
// index.js
const { run } = require("@probot/github-action");
const app = require("./app");

run(app).catch((error) => {
  console.error(error);
  process.exit(1);
});
```

For an example Probot App published as GitHub Action, see https://github.com/probot/example-github-action#how-it-works

## How it works

[Probot](https://probot.github.io/) is a framework for building [GitHub Apps](docs.github.com/apps), which is different to creating [GitHub Actions](https://docs.github.com/actions/) in many ways, but the functionality is the same:

Both get notified about events on GitHub, which you can act on. While a GitHub App gets notified about a GitHub event via a webhook request sent by GitHub, a GitHub Action can receive the event payload by reading a JSON file from the file system. We can abstract away the differences, so the same hello world example app shown above works in both environments.

Relevant differences for Probot applications:

1. You cannot authenticate as the app. The `probot` instance you receive is authenticated using a GitHub token. In most cases the token will be set to `secrets.GITHUB_TOKEN`, which is [an installation access token](https://docs.github.com/en/actions/reference/authentication-in-a-workflow#about-the-github_token-secret). The provided `GITHUB_TOKEN` expires when the job is done or after 6 hours, whichever comes first. You do not have access to an `APP_ID` or `PRIVATE_KEY`, you cannot create new tokens or renew the provided one.
2. `secrets.GITHUB_TOKEN` is scoped to the current repository. You cannot read data from other repositories unless they are public, you cannot update any other repositories, or access organization-level APIs.
3. You could provide a personal access token instead of `secrets.GITHUB_TOKEN` to workaround the limits of a repository-scoped token, but be sure you know what you are doing.
4. You don't need to configure `WEBHOOK_SECRET`, because no webhook request gets sent, the event information can directly be retrieved from environment variables and the local file system.

For a more thorough comparison, see [@jasonetco]() posts:

1. [Probot App or GitHub Action](https://jasonet.co/posts/probot-app-or-github-action/) (Jan 2019)
2. [Update from April 2020](https://jasonet.co/posts/probot-app-or-github-action-v2/)

## License

[ISC](LICENSE)
