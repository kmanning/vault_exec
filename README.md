# vault_exec
vault_exec is a wrapper to execute arbitrary scripts using temporary credentials managed by Vault.  Vault is an application written by Hashicorp, that can be used to manage secrets.  Learn more about Vault [here](https://www.vaultproject.io/).

# Assumptions

- The vault client is already installed on your machine.  Instructions are available [here](https://www.vaultproject.io/docs/install/index.html).
- You have valid vault credentials to create temporary AWS keys, and know the vault path to do this.  Instructions are available [here](https://www.vaultproject.io/docs/secrets/aws/index.html)
- The vault_exec currently only runs on a *nix system.

# Getting Started

Optionally set the location of your vault instance.  If you do not specify this, it will use Vault's default (https://127.0.0.1:8200/v1/):

```
# Optionally add this to your profile script so it's always available
> export VAULT_ADDR=https://vault.mycompany.com/
```

Set the Vault path to create temporary AWS credentials.  If you don't specify this, a default will be provided ('aws/creds/developer'):

```
# Optionally add this to your profile script so it's always available
> export VAULT_CREDENTIALS_PATH=aws/creds/my_creds_path
```

Set the length of time you'd like your credentials to be valid for as a golang time string (``^[0-9]+(s|m|h)$``), from "1h" to "12h". If you don't specify this, the default of "1h" (one hour) will be used:

```
# Request 12-hour credentials. Optionally add this to your profile script so it's always available
> export VAULT_CREDENTIALS_DURATION=12h
```

Authenticate against your vault instance, using your preferred method.  This example uses LDAP.

```
> vault auth -method=ldap username=<YOUR USERNAME>
```

Add the vault_exec script to your PATH through your preferred method.  This example places it in /usr/local/bin/.

```
> sudo cp ./bin/vault_exec /usr/local/bin/
```

Execute your command with vault_exec - temporary credentials will be provided in environment variables AWS_ACCESS_KEY_ID and AWS_SECRET_KEY:

```
> vault_exec 'echo AWS_ACCESS_KEY: $AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY: $AWS_SECRET_ACCESS_KEY'

Vault credentials file not found - creating new credentials.
AWS_ACCESS_KEY_ID: ABCDEFG, AWS_SECRET_ACCESS_KEY: 1234567890
```

Your credentials will be stored in ~/.aws/vault_aws_credentials.  If they have not expired, they will be reused between runs.

```
> vault_exec 'echo AWS_ACCESS_KEY: $AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY: $AWS_SECRET_ACCESS_KEY'

Vault credentials will remain valid for: 999 seconds
AWS_ACCESS_KEY_ID: ABCDEFG, AWS_SECRET_ACCESS_KEY: 1234567890
```

If your credentials have expired, they will automatically be renewed.

```
> vault_exec 'echo AWS_ACCESS_KEY: $AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY: $AWS_SECRET_ACCESS_KEY'

Vault credentials have expired or are not valid - creating new credentials.
AWS_ACCESS_KEY_ID: GFEDCBA, AWS_SECRET_ACCESS_KEY: 0987654321
```

Set DEBUG_VAULT_EXEC=true to debug the script:

```
> DEBUG_VAULT_EXEC=true vault_exec echo hello

VAULT_CREDENTIALS_PATH=aws/creds/developer
Vault credentials will remain valid for: 3286 seconds
AWS_ACCESS_KEY_ID=ABCDEFG
AWS_SECRET_ACCES_KEY=1234567890
hello
```
