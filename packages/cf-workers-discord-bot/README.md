# Cloudflare Workers Discord Bot

Interact with [Discord](https://discord.com/) from within [Cloudflare Workers](https://workers.cloudflare.com/).

_This is the same as @glenstack/cf-workers-fetch-helpers, except it has the [Buffer bug](https://github.com/glenstack/glenstack/issues/13#issuecomment-1075535069) fixed_

## Installation

```sh
npm install --save @code6226/cf-workers-discord-bot
```

## Usage

### `createSlashCommandHandler`

```typescript
import {
  createSlashCommandHandler,
  ApplicationCommand,
  InteractionHandler,
  Interaction,
  InteractionResponse,
  InteractionResponseType,
} from "@code6226/cf-workers-discord-bot";

const helloCommand: ApplicationCommand = {
  name: "hello",
  description: "Bot will say hello to you!",
};

const helloHandler: InteractionHandler = async (
  interaction: Interaction
): Promise<InteractionResponse> => {
  const userID = interaction.member.user.id;

  return {
    type: InteractionResponseType.ChannelMessageWithSource,
    data: {
      content: `Hello, <@${userID}>!`,
      allowed_mentions: {
        users: [userID],
      },
    },
  };
};

const slashCommandHandler = createSlashCommandHandler({
  applicationID: "799627301675466772",
  applicationSecret: APPLICATION_SECRET, // You should store this in a secret
  publicKey: "1b780f7f71ac39645d44cc4dce8fa78c860d0157cb0912d755b7a7cb95394532",
  commands: [[helloCommand, helloHandler]],
});

addEventListener("fetch", (event) => {
  event.respondWith(slashCommandHandler(event.request));
});
```

`createSlashCommandHandler` takes one parameter:

1. An object with the following parameters:

   - `applicationID`: The "Client ID" of your [Discord application](https://discord.com/developers/applications/).
   - `applicationSecret`: The "Client Secret" of your [Discord application](https://discord.com/developers/applications/). **Note: this should be stored securely as a [secret](https://developers.cloudflare.com/workers/cli-wrangler/commands#secret) in the Worker.**
   - `publicKey`: The "Public Key" of your [Discord application](https://discord.com/developers/applications/).
   - `commands`: An array of the commands for your bot to register and respond to.

     Each element of this array should be itself an array with two elements:

     1. An [`ApplicationCommand`](https://discord.com/developers/docs/interactions/slash-commands#applicationcommand) object used to register the command.
     1. A function which takes a single parameter, an [`Interaction`](https://discord.com/developers/docs/interactions/slash-commands#interaction) object, and returns an [`InteractionResponse`](https://discord.com/developers/docs/interactions/slash-commands#interaction-response) or a Promise of an [`InteractionResponse`](https://discord.com/developers/docs/interactions/slash-commands#interaction-response).

It returns a function which takes a Request and returns a Promise of a Response. It should typically be given to the `event.respondWith` function.

This makes your application respond to three types of Requests:

- `GET /`: Redirects the user to Discord to authorize the bot on a server.
- `POST /interaction`: The incoming webhook from Discord to respond to Slash Commands or Pings. **Note: this URL needs to be configured in the Discord Developer Portal as the "Interactions Endpoint URL" e.g. `https://my-discord-bot.workers.dev/interaction`**
- `GET /setup`: Registers the commands with Discord. **Note: this endpoint must be visited once to initialize the new commands, and again everytime the commands are updated.**
