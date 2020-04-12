# Notes

My notes on testing using schemathesis.

## Builder

1. Installed ruby.
1. Ran `bundle install`. Got a coffee, as it takes forever on Windows.
1. Created and activated a python virtual environment using `virtualenv`.
1. Determined that the run command was `node server.js` using scripts in `package.json`.
1. Determined the host and port by reading the source code, which makes the [default port `3000` and the default IP `0.0.0.0` in `server.js`](./index.js). This is also printed to the console when the app starts.
1. Found no OpenAPI spec. Determined the routes by following an `app.use` statement importing a router. `./app/router/index.js`. From here, each route has its own particularities. For example, `/facts` is backed by `./app/router/fact.routes.js`.
1. I used Stoplight Studio to document three methods of the API. This took about 10 minutes, and is probably faster once you get the hang of it. I was able to make the spec by mixing the documentation, which showed the return values, and the source, which showed the query parameters and endpoints.
1. When trying to start the server, got a `throw new Error('\'clientAccessToken\' cannot be empty.');` error. Tracing back from the error, I found that `keys.js` needed `APIAI_ACCESS_TOKEN` defined in the environment. Other things I needed to define were `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET` and `BASE_URL`. After this, the server starts.
1. In a few seconds, the server crashes because it cannot access mongodb. Tried `docker run -d -p 27017:27017 --name mongo1 mongo` but it didn't work because the url was hardcoded as `mongodb://${this.username}:${this.password}@ds157298.mlab.com:57298/${this.name}` in `keys.js`. Changed it to `mongodb://localhost:27017/cat-facts`. Still didn't work. Googling around, I found [this](https://stackoverflow.com/questions/58160691/server-selection-timed-out-after-10000-ms-cannot-connect-compass-to-mongodb-on), which suggested using `127.0.0.1` instead. Still didn't work. Continuing to Google around, I found the suggestion to [set `useUnifiedToplogoy` to false](https://github.com/Automattic/mongoose/issues/8381). Still didn't work. After digging through the codebase, I realized that the url to `mongoose` is supplied _twice_ in `server.js`, once to `MongoStore` and once to `mongoose.connect`. The version supplied to `mongoose.connect` was a hardcoded URL. The author must have copied and pasted it from keys.js. After I changed the hardcoded version to use url defined in `keys.js`, it worked!

In the future, it would be interesting to mock more endpoints to see if a bug could be provoked.

## Runner

## Mocker