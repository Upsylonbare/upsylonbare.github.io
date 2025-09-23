## How to send emails with gmail, using XOAUTH2, [msmtp](https://marlam.de/msmtp/), and [oama](https://github.com/pdobsan/oama)

To contribute to Linux Kernel and sending my patches to the mailing list, I need to be able to send email using my gmail account. Google has disabled the "less secure apps" option, so I had to find a way to authenticate using XOAUTH2.

Here is how I did it.

---

### Getting the OAuth2 tokens

First, you need to create a project in the [Google Cloud Console](https://console.cloud.google.com/).

#### 1. Create a new project.

Use a name like "upstream patches" or something that makes sense to you.

#### 2. Configure the authentication to google services.

In the search bar, type "Google Auth Platform" and select it.
Then click on get started in the middle of the page.

<p align="center">
    <img src="/res/GetStartedGoogleAuth.png" alt="Screenshot of the Google Auth Platform page" width="75%">
</p>

#### 3. Fill in the project configuration.

You need to fill in the following fields:

- App name: "**oama**"
- User support email: **youremail@ddr.ess**

Next.

- Audience: "**External**" (I don't think it matters much)

Next.

- Contact Information: **youremail@ddr.ess**

Next.

- Tick to agree to the Google API services user data policy.

Continue.

🪄 Create 🪄

#### 4. Create OAuth Client

<p align="center">
    <img src="/res/CreateAuthClientMetrics.png" alt="Screenshot of the OAuth Client creation page" width="75%">
</p>

Fill in the following fields:

- Application type: "**Desktop app**"
- Name: "**oama**"

Create.

A popup will appear with your Client ID and Client Secret. Copy them somewhere safe or download the JSON file.

<p align="center">
  <img src="/res/GOAUTHTokens.png" alt="Screenshot of the popup with the Client ID and Client Secret" width="50%">
</p>

### Configure [oama](https://github.com/pdobsan/oama)

Now that you have your Client ID and Client Secret, you can configure oama to get your access and refresh tokens.

Read the documentation as it is always useful to learn more about the tool you'll use.

Install it using your package manager or compile it from source.

Launch oama:

```bash
$> oama
WARNING -- Could not find config file: $HOME/.config/oama/config.yaml
Creating initial config file ...
... done.
Edit it then start oama again.
```

Then edit the config file at `$HOME/.config/oama/config.yaml` and fill in the `client_id` and `client_secret` fields with the values you got from the Google Cloud Console.

```yaml
## oama version 0.22.0 - 2025-08-29 0290.e419ef10
## This is a YAML configuration file, indentation matters.
## Double ## indicates comments while single # default values.
## Not all defaults are shown, for full list run `oama printenv`
## and look at the `services:` section.

## Possible options for keeping refresh and access tokens:
## GPG - in a gpg encrypted file $XDG_STATE_HOME/oama/<email-address>.oauth
##       (XDG_STATE_HOME defaults to ~/.local/state)
## GPG - in a gpg encrypted file ~/.local/state/oama/<email-address>.oauth
## KEYRING - in the keyring of a password manager with Secret Service API
##
## Choose exactly one.

encryption:
    tag: KEYRING

# encryption:
#   tag: GPG
#   contents: your-KEY-ID

## Builtin service providers
## - google
## - microsoft
## Required fields: client_id, client_secret
##
services:
  google:
    client_id: application-CLIENT-ID
    client_secret: application-CLIENT-SECRET
  ## Alternatively get them from a password manager using a shell command.
  ## If both variants are present then the _cmd versions get the priority.
  ## For example:
  # client_id_cmd: |
  #   pass email/my-app | head -1
  # client_secret_cmd: |
  #   pass email/my-app | head -2 | tail -1
  #  auth_scope: https://mail.google.com/

  microsoft:
     client_id: application-CLIENT-ID
  ## client_secret is not needed for device code flow
  #  auth_endpoint: https://login.microsoftonline.com/common/oauth2/v2.0/devicecode
  ##
  ## client_secret might be needed for other authorization flows
  #  client_secret: application-CLIENT_SECRET
  ## auth_endpoint: https://login.microsoftonline.com/common/oauth2/v2.0/authorize
  #
  #  auth_scope: https://outlook.office.com/IMAP.AccessAsUser.All
  #     https://outlook.office.com/SMTP.Send
  #     offline_access
  #  tenant: common

  ## User configured providers
  ## Required fields: client_id, client_secret, auth_endpoint, auth_scope, token_endpoint
  ##
  ## For example:
  # yahoo:
  #   client_id: application-CLIENT-ID
  #   client_id_cmd: |
  #     password manager command ...
  #   client_secret: application-CLIENT_SECRET
  #   client_secret_cmd: |
  #     password manager command ...
  #   auth_endpoint: EDIT-ME!
  #   auth_scope: EDIT-ME!
  #   token_endpoint: EDIT-ME!
```

Note the Google service part

```yaml
services:
  google:
    client_id: application-CLIENT-ID
    client_secret: application-CLIENT-SECRET
```

Fill in the `client_id` and `client_secret` fields with your values.

If you don't want to store your client secret in plain text, you can use a password manager and use the `_cmd` fields to get them, more details [here](https://github.com/pdobsan/oama#application-client_id-and-client_secret)

Now run oama with your email address:

```bash
$> oama authorize google <youremail@ddr.ess> --nohint
Authorization to grant OAuth2 access to youremail@ddr.ess started ... 
Visit http://localhost:50589/start in your browser ...
```

This will open a link in your browser, asking you to authorize oama to access your gmail account.

<p align="center">
    <img src="/res/GoogleWarning.png" alt="Screenshot of the unverified app warning" width="75%">
</p>
You may have a warning about the app not being verified, just click on "Advanced" and then "Go to oama (unsafe)".

Then click on "Allow".

🪄 Now OAMA should be able to authenticate to Google using OAUTH2 🪄

### Configure [msmtp](https://marlam.de/msmtp/)

Now that you have your refresh and access tokens, you can configure msmtp to use them to send emails.

In your home directory, create a file named `.msmtprc` with the following content:

```txt
defaults
auth	oauthbearer
tls		on
tls_certcheck	on
tls_trust_file /etc/ssl/certs/ca-certificates.crt
logfile		~/.msmtp.log

account		gmail
host		smtp.gmail.com
port		587
from		<youremail@ddr.ess>
user		<youremail@ddr.ess>
passwordeval	oama access <youremail@ddr.ess>

account default : gmail
```

🪄 Everything should now work 🪄