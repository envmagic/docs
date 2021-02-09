# envmagic

> A Secrets Manager That Never Leaves Your Private Cloud

## Features

- Persists your app secrets/config to your AWS Parameter Store
- Pushes updated secrets/config to your server process instantly
- Audit log/version history from AWS Parameter Store
- Self-hosted or SaaS option (either way, secrets stay in your own AWS account)

## AWS Account / Organization Setup

This only needs to be done once for your entire organization.

1. Sign up at https://envmagic.com
2. Note your API key, you will use it in a later step
3. Install the `envmagic` CLI as described in [the installation instructions](#install-the-cli) below
4. Run `envmagic setup`, paste the API key from step 2, and follow the rest of the prompts

## Install the CLI

This needs to be done on every developer machine, or server/container where your app runs and needs to pull secrets from.

```
sudo curl -L "https://github.com/envmagic/cli/releases/download/1.0.0/envmagic-$(uname -s)-$(uname -m)" -o /usr/local/bin/envmagic
chmod +x /usr/local/bin/envmagic
```

For an Intel Mac x86_64 machine, for example, the command would execute:

```
sudo curl -L "https://github.com/envmagic/cli/releases/download/1.0.0/envmagic-Darwin-x86_64" -o /usr/local/bin/envmagic
chmod +x /usr/local/bin/envmagic
```

## CLI Usage

### Setting an app's environment variables

To create/update a project, environment, and set the .env file variables required to run it, simply run:

```
envmagic
```

and after choosing the appropriate prompts

```console
$ envmagic
Project: my-app  ✔
Environment: prod  ✔
Action: ❯ Edit .env file
```

it will present you with a virtual .env file that you can set all your environment variables on:

```ini
# This is the virtual .env file for my-app in the prod environment.
# Specify environment variables using KEY=value123 notation.
# HINT: Use vi shortcuts. Press [i] to enter insert mode, then [esc] :wq to save & quit.
MY_API_KEY=abcdef123
PORT=9001
```

### Running an app

To run your app using the virtual .env file we just created, simply prepend `envmagic --project my-app --env prod --` to your app start command.

For example, if you use `python app.py` to start your app, simply run:

```
envmagic --project my-app --env prod -- python app.py
```

Note that if you don't specify a project and/or environment on the command line, `envmagic` will prompt you to select one using a convenient, filterable dropdown:

```console
$ envmagic
Project: Search projects...
         ❯ my-app
           another-app
Environment: Search environments...
```

While your app is running, if you or someone else in your organization runs `envmagic` and changes an environment variable for the above project & env, your app will restart and instantly pick up the new value for the environment variable.

## Usage with Docker / Docker Compose

### 1. Update your `Dockerfile`

Edit your `Dockerfile` to install the `envmagic` CLI and prepend it to your existing CMD, like so:

```Dockerfile
# Dockerfile

# add this below your OS package installs,
RUN curl -L "https://github.com/envmagic/cli/releases/download/1.0.0/envmagic-$(uname -s)-$(uname -m)" -o /usr/local/bin/envmagic
RUN chmod +x /usr/local/bin/envmagic
# but above your application dependency install steps...

# finally, use envmagic to inject env vars. NOTE that using the array style CMD [] is recommended over the string style
CMD ['/usr/local/bin/envmagic', '--project', 'my-app', '--env', 'prod', '--', './start-app.sh']
```

### 2. Override the `docker run --command` you use for local development

```
$ docker run [other options] --entrypoint '/bin/sh' $IMAGE -c '/usr/local/bin/envmagic --project my-app --env dev -- ./start-app.sh'
```

### 3. Or override the `docker-compose.yml` command you use for local development

```yml
# docker-compose.yml

version: "3.9"
services:
  # other services

  # the service that uses envmagic
  myapp:
    # only if you need to override the default
    # entrypoint: /bin/sh

    # Note how it uses `--env dev` instead of prod
    command:
      - /usr/local/bin/envmagic
      - --project
      - my-app
      - --env
      - dev
      - --
      - ./start-app.sh
```

## Troubleshooting

### Missing AWS Region

> ERROR: AWS region is not set! Please set it using AWS_REGION or --region

The `envmagic` CLI needs to know which AWS region to store/look for your app environment variables in.

You can configure this by either setting the `AWS_REGION` or `AWS_DEFAULT_REGION` environment variable, e.g.

```
AWS_REGION=us-east-1 envmagic
```

Or by using the `--region` flag,

```
envmagic -r us-east-1
```

which will override any `AWS_REGION` set in your shell or default region set in `~/.aws/config`.

### Missing AWS Credentials

> ERROR: AWS credentials are missing! Please provide them using one of the available methods

The `envmagic` CLI could not detect any valid AWS credentials. You can either supply them via [environment variables](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-envvars.html) (`AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`) or some other method (EC2 IAM roles).

If running on an EKS/Kubernetes Cluster, we recommend using [kube2iam](https://github.com/jtblin/kube2iam) to generate per-service, ephemeral credentials that are automatically rotated.

If running on a developer machine, it's a good security practice to use [aws-vault](https://github.com/99designs/aws-vault#quick-start) to store the credentials in your OS keychain and generate short-lived credentials upon authentication. Thus, if the short-lived credentials leak, they will automatically expire soon and you won't have to worry about invalidating your keys.

### EnvMagic Not Set Up

> ERROR: EnvMagic is not set up in this AWS account or region.

This error can happen for any of the following reasons:

1. You forgot to run `envmagic setup`. Note that this only has to be done once for the entire AWS account/organization.
2. `envmagic` is not set up in this AWS region. Double-check `$AWS_REGION` or specify it via the CLI with `envmagic --region us-east-1`, etc.
3. `envmagic` is not set up in the AWS account you're using. Double-check that you're using the correct `$AWS_ACCESS_KEY_ID`, `$AWS_PROFILE`, or default `~/.aws/credentials`.
