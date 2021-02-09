# envmagic

> A Secrets Manager That Never Leaves Your Private Cloud

## Features

- Persists your app secrets/config to AWS Parameter Store
- Pushes updated secrets/config to your server process instantly
- Audit log/version history from AWS Parameter Store
- Self-hosted or SaaS option (either way, secrets stay in your own AWS account)

## One-Time Setup

This only needs to be done once for your entire organization.

First, sign up at https://envmagic.com, get your API key, and replace `__API_KEY_HERE__` below with it. Also replace `YOUR_REGION` with your primary AWS region, e.g. `us-east-1`.

Note that these commands use the AWS CLI to set your envmagic config in AWS Parameter Store. If you don't have the AWS CLI yet, you must [install](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) and [configure it with your AWS credentials](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html) before continuing.

```
aws ssm get-parameter --region YOUR_REGION --name "/envmagic/installations/default" --type "SecureString" --value "saas"
aws ssm get-parameter --region YOUR_REGION --name "/envmagic/installations/saas/eventsUrl" --type "SecureString" --value "wss://events.envmagic.com"
aws ssm get-parameter --region YOUR_REGION --name "/envmagic/installations/saas/apiKey" --type "SecureString" --value "__API_KEY_HERE__"
```

If you make a mistake and have to change/rewrite the above values, add the `--overwrite` flag to each command.

## Install

Needs to be done on every developer machine, or server/container where your app runs and needs to pull secrets from.

```
$ sudo curl -L "https://github.com/envmagic/cli/releases/download/1.0.0/envmagic-$(uname -s)-$(uname -m)" -o /usr/local/bin/envmagic
```

For an Intel Mac x86_64 machine, for example, the command would execute:

```
$ sudo curl -L "https://github.com/envmagic/cli/releases/download/1.0.0/envmagic-Darwin-x86_64" -o /usr/local/bin/envmagic
```

## Usage

### configure

To configure your first project, environment, and set the .env file variables required to run it, simply run:

```
$ envmagic
```

and after choosing the appropriate prompts

```
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

### run

To run your app using the virtual .env file we just created, simply prepend `envmagic --project my-app --env prod --` to your app start command. For example, if you use `python app.py` to start your app, you would run:

```
$ envmagic --project my-app --env prod -- python app.py
```

Or the short flags,

```
$ envmagic -p my-app -e prod -- python app.py
```

While your app is running, if you or someone else in your organization runs `envmagic` and changes an environment variable for the above project & env, your app will restart and instantly pick up the new value for the environment variable.

## Troubleshooting

### ERROR: AWS region is not set! Please set it using AWS_REGION or --region

The `envmagic` CLI needs to know which AWS region to store/look for your app environment variables in.

You can configure this by either setting the `AWS_REGION` or `AWS_DEFAULT_REGION` environment variable, e.g.

```
$ AWS_REGION=us-east-1 envmagic
```

Or by using the `--env` flag,

```
$ envmagic -e us-east-1
```

which will override any `AWS_REGION` set in your shell or default region set in `~/.aws/config`.

### ERROR: AWS credentials are missing! Please provide them using one of the available methods

The `envmagic` CLI could not detect any valid AWS credentials. You can either supply them via [environment variables](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-envvars.html) (`AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`) or some other method (EC2 IAM roles).

If running on an EKS/Kubernetes Cluster, we recommend using [kube2iam](https://github.com/jtblin/kube2iam) to generate per-service, ephemeral credentials that are automatically rotated.

If running on a developer machine, it's a good security practice to use [aws-vault](https://github.com/99designs/aws-vault#quick-start) to store the credentials in your OS keychain and generate short-lived credentials upon authentication. Thus, if the short-lived credentials leak, they will automatically expire soon and you won't have to worry about invalidating your keys.
