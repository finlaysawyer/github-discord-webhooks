[![pre-commit.ci status](https://results.pre-commit.ci/badge/github/finlaysawyer/github-discord-webhooks/master.svg)](https://results.pre-commit.ci/latest/github/finlaysawyer/github-discord-webhooks/master)

# github-discord-webhooks

A serverless function running on [Cloudflare Workers](https://workers.cloudflare.com/) and [D1](https://developers.cloudflare.com/d1/) that delivers enhanced GitHub Workflow webhooks for Discord.

## Features

- More information about the running Workflow, author and commit information
- Messages are edited as workflow events come through, reducing spam:

  ![demo](https://user-images.githubusercontent.com/18363677/209872272-5b7324bf-92f6-4380-b5a5-f503053ae73e.gif)

# Installation

If you want to run and/or deploy this project for yourself, you will need the following:

- [wrangler](https://developers.cloudflare.com/workers/get-started/guide#1-install-wrangler-workers-cli) CLI
- Node.js LTS

## 1. Create a Discord Webhook and Secret

Create a Discord Webhook in the channel of your choice following the instructions [here](https://support.discord.com/hc/en-us/articles/228383668-Intro-to-Webhooks).

Copy `.dev.vars.example` to `.dev.vars` and update the `DISCORD_WEBHOOK` variable with your webhook URL. This value will be used for local development.

Create a secret for use in the deployed version:

```console
$ wrangler secret put DISCORD_WEBHOOK
 ⛅️ wrangler 2.6.2
-------------------
Enter a secret value: *************************************************************************************************************************
🌀 Creating the secret for the Worker "github-webhooks-worker"
```

## 2. Create a D1 Database

###### NOTE: D1 is currently in Alpha, the following is subject to change

This project uses a [D1 Database](https://blog.cloudflare.com/introducing-d1/) to track the state of
the Discord message associated with a GitHub workflow run. To generate a database for the project, use the following:

```console
$ wrangler d1 create webhook_state
...
✅ Successfully created DB 'webhook_state'!

Add the following to your wrangler.toml to connect to it from a Worker:

[[ d1_databases ]]
binding = "DB" # i.e. available in your Worker on env.DB
database_name = "webhook_state"
database_id = "48298b61-95b1-4705-b948-ebc87c434246"
```

Then, copy the `wrangler.example.toml` file into `wrangler.toml`, and paste in the suggested snippet to create your
database binding.

Next, populate your local database with the schema:

```console
$ wrangler d1 execute DB --local --file=./schema.sql
🌀 Mapping SQL input into an array of statements
🌀 Loading DB at .wrangler/state/d1/DB.sqlite3
```

And finally, populate the deployed database with the schema (by dropping the `--local` flag):

```console
$ wrangler d1 execute DB --file=./schema.sql
🌀 Mapping SQL input into an array of statements
🌀 Parsing 2 statements
🌀 Executing on DB (48298b61-95b1-4705-b948-ebc87c434246):
🚣 Executed 2 commands in 41.074951000511646ms
```

More documentation on getting started with D1 can be found [here](https://developers.cloudflare.com/d1/get-started/).

## 4. Running locally

To run the project locally, use the start npm script:

```console
$ npm run start

> github-webhooks-worker@0.0.0 start
> wrangler dev --persist --local
...
[mf:inf] Worker reloaded! (9.09KiB)
[mf:inf] Listening on 0.0.0.0:8787
```

## 5. Deploy to Cloudflare

To deploy to Cloudflare, use the deploy npm script:

```console
$ npm run deploy

> github-webhooks-worker@0.0.0 deploy
> wrangler publish
...
Uploaded github-webhooks-worker (2.00 sec)
Published github-webhooks-worker (1.56 sec)
```
