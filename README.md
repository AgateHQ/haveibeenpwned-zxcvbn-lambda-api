# **Your** 5-Min. Secure Password Scoring and Pwnage Protection API
Deploy your very own serverless API on AWS Lambda to score users' new passwords with Dropbox's fantastic `zxcvbn` library and anonymously (range) search for matches in Troy Hunt's [haveibeenpwned](https://haveibeenpwned.com/) `pwnedpasswords` database.

![API in Action](.github/pwnage.gif?raw=true "API in Action")

&nbsp;&nbsp;&nbsp;&nbsp;*Example: handling results with [VuetifyJS](https://github.com/vuetifyjs/vuetify)*
<br>
<br>
## Motivation
<a href="https://twitter.com/DetroitEnglish/status/1008276231199055874" target="_blank">People seemed to think this concept was neat</a>. AWS Lambda was the ~~best~~ only solution I could think of that's easy to deploy, programming language agnostic and, most importantly, **impossible to implement without https**.

---

## Quick Start
1. Create an AWS profile with IAM full access, Lambda full access and API Gateway Administrator privileges.
2. Add the keys to your ~/.aws/credentials file:
    ```
    [pwnage]
    aws_access_key_id = YOUR_ACCESS_KEY
    aws_secret_access_key = YOUR_ACCESS_SECRET
    ```
    To use another profile, set it with `npm config set haveibeenpwned-zxcvbn-lambda-api:aws_profile some-aws-profile`  (default: `pwnage`)

1. Rename `example.env.json` to `env.json` and configure as you see fit. Note that all entries **must** be strings, less we anger the Lambda gods.
2. Set your AWS region of choice with `npm config set haveibeenpwned-zxcvbn-lambda-api:aws_region some-aws-region` (default: `eu-central-1`)
3. Set deployment AWS API Gateway environment (i.e. version) with `npm config set haveibeenpwned-zxcvbn-lambda-api:aws_environment staging` (default: `development`)
4. Install dependencies with `npm install`
5. Launch 🚀 with `npm run deploy`
6. Change whatever you need to change in the AWS API Gateway, DNS, etc to make this work with your own application.

### Configuration
The following options are configurable via `env.json`:

- `"ALLOW_ORIGINS"`: A **comma-separated** whitelist of origins for Cross Origin Resource Sharing. If none are provided, all origins are allowed (default: `""`)
    - Example: `"ALLOW_ORIGINS": "https://secure.domain.lol,http://unsecure.domain.wtf"`

- `"CORS_MAXAGE"`: Value in seconds for the `Access-Control-Max-Age` CORS header (default: `"0"`)

- `"ALWAYS_RETURN_SCORE"`: Return the `zxcvbn` score even if the `pwnedpasswords` match value is > 0. See [The Response](##The-Response) for details (default: `"false"`)

Note that all `env.json` values **must** be strings, less you anger the Lambda gods.

### Updating
Update the Lambda API with any changes you make to the source by running `npm run update`.

Update environment variables à la changes to `env.json` by running `npm run update-env`.

### Sorcery
Deployment's automated using the cool-as-a-cucumber [claudia.js](https://claudiajs.com/documentation.html) for end-to-end Lambda and API Gateway configuration - please refer to the claudia.js docs to learn more about this serverless voodoo magic.

## REST API

Following successful deployment or update, `claudia.js` prints the AWS config for your freshly deployed Lambda function, including an https url for instant and secure access to your function:

GET the healthcheck/container-warmup endpoint:
```
    https://{{GENERATED_ID}}.execute-api.{{AWS_REGION}}.amazonaws.com/production/{{ROUTE_PREFIX}}/_up
```

POST user password input as JSON to:
```
    https://{{GENERATED_ID}}.execute-api.{{AWS_REGION}}.amazonaws.com/production/{{ROUTE_PREFIX}}/{{SCORING_ENDPOINT}}
```

### Request

POST user password input as JSON with field `password` like so:
```
{
  "password": "monkey123"
}
```

### Response

The API will reply with an appropriate status code and return JSON with `ok` indicating successful scoring and range search, a strength estimation `score` of 0 through 4 per `zxcvbn`, and `pwned` matches, indicating the number times the input appears in the `haveibeenpwned` database.

```
{
    "ok": true,
    "score": 0,
    "pwned": 56491
}
```
By default, if `pwned` is greater than 0, then `score` will **always** be 0. You can override this behavior by settings `"ALWAYS_RETURN_SCORE"` to `"true"` in `env.json`

#### Errors

Failure will return JSON to inform you that something's not `ok` and a `message` as to why.

```
{
    "ok": false,
    "message": "Something went kaput! :( "
}
```

### Good to Know
The health-check endpoint `/_up` is included by default; this also serves as a handy means to warm-up a Lambda function container before your users start feeding you input.

Lambda's Node 8 runtime supports `async/await` natively, nevertheless deploying will transpile `src/index.js` to JS compatible with Node 6.

Finally, it may seem weird that deploying will first nuke `package-lock.json` and then reinstall deps - it is a workaround to avoid claudia.js weirdness that I cannot explain but occurs when retrying failed deployments ¯\\\_(ツ)\_/¯

## Because Software

### Disclaimer
I am not affiliated with Amazon, Troy Hunt, Dropbox, haveibeenpwned, good software development in general, or any combination thereof.

Handling user passwords is no laughing matter, so handle them with care and respect.

Just like your own users, assume that I have no idea what I'm doing. This part is important, because I have no idea what I'm doing.

**REVIEW THE SOURCE**, and use at your own risk!

### License
MIT
