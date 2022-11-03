# Queuing Starter Kit for JavaScript [<img align="right" alt="Deploy to Fastly" src="https://deploy.edgecompute.app/button"/>](https://deploy.edgecompute.app/deploy)

Park website visitors in a virtual queue to reduce the demand on your origins during peak times.

**For more details about other starter kits for Compute@Edge, see the [Fastly developer hub](https://developer.fastly.com/solutions/starters)**

## Features

- Park visitors in a queue to limit the amount of active users on the website ⏳
- Ship queue analytics to [log endpoints](https://developer.fastly.com/learning/integrations/logging) 🔎
- Allow certain requests such as robots.txt, favicon to bypass the queue 🤖
- No client-side scripting required ⚡️

## Getting started

1. If you haven't already, [sign up for Upstash](https://www.npmjs.com/package/@upstash/redis) and create a Redis service.
2. Initialize a Compute@Edge project using this starter kit.
   ```sh
   fastly compute init --from=https://github.com/sohsoh5/Javascript-Waitroom
   ```
   
3. Run `fastly compute publish` to create a service and perform #4, #5, and #6 
   - You can also manually alter your fastly.toml file to include and already existing service ID and enter the backends/dictionaries.
4. Create the `upstash` backend, changing the default hostname to the one provided in the Upstash console.
5. Create the `protected_content` backend by accepting the default example host or setting your own.
6. Populate the `config` and `waitroom_param` dictionaries by following the prompts to configure Upstash and set a secret for signing cookies.
7. Run `fastly compute publish` once more to deploy your queue.

## Understanding the code

This starter is fully-featured, and requires some dependencies on top of the [`@fastly/js-compute`](https://www.npmjs.com/package/@fastly/js-compute) npm package. The following is a list of the dependencies used:

- **[@upstash/redis](https://www.npmjs.com/package/@upstash/redis)** - a REST-based Redis client for storing queue state. You could easily swap this for your own storage backend.
- **[jws](https://www.npmjs.com/package/jws)** - a library for generating and validating [JSON Web Tokens](https://datatracker.ietf.org/doc/html/rfc7519).

The starter will require a backend to be configured to send requests to once visitors have made it through the queue. For demonstration, the default is a public S3 bucket with some assets and an index page.

The template uses webpack to bundle `index.js` and its imports into a single JS file, `bin/index.js`, which is then wrapped into a `.wasm` file, `bin/index.wasm` using the `js-compute-runtime` CLI tool bundled with the `@fastly/js-compute` npm package, and bundled into a `.tar.gz` file ready for deployment to Compute@Edge.

## Design

When a visitor makes a request for the first time, we generate a signed JWT containing their _position_ in the queue, which is determined by fetching the current queue _length_ from Redis (`INCR queue:length`). This signed JWT is sent back to the visitor as an HTTP cookie.

On a regular basis, the current queue _cursor_ is updated in Redis (`INCR queue:cursor`). This effectively lets a user in. To show a visitor how many others are in front of them in the queue, we subtract the current queue _cursor_ from the _position_ saved in their JWT.

On subsequent requests, when a JWT is supplied, we verify the signature and extract the _position_ from the JWT. If the current queue _cursor_ is higher than the user's signed _position_, they will be allowed in.

## Security issues

Please see our [SECURITY.md](SECURITY.md) for guidance on reporting security-related issues.
