# actions-demo

This is a demo of how to use GitHub Actions with the Snowflake CLI. This uses a Snowflake connection named `ga_dev_keypair_auth` defined in the `.snowflake/config.toml` file. The connection only defines a user, role and authenticator.

## Snowflake CLI Connection Configuration

```toml
default_connection_name = "ga_dev_keypair_auth"

[connections.ga_dev_keypair_auth]
authenticator = "SNOWFLAKE_JWT"
user = "ga_dev"
role = "github_action_dev"
```

We are using [key-pair authentication](https://docs.snowflake.com/en/user-guide/key-pair-auth) to authenticate with this connection. The Snowflake account and private key are stored as secrets in the GitHub repository and referenced in the workflow as part of the environment when executing Snowflake CLI commands.

```yml
        # Use the CLI
      - name: Execute Snowflake CLI command
        env:
          SNOWFLAKE_CONNECTIONS_GA_DEV_KEYPAIR_AUTH_ACCOUNT: ${{ secrets.SNOWFLAKE_ACCOUNT }}
          SNOWFLAKE_CONNECTIONS_GA_DEV_KEYPAIR_AUTH_PRIVATE_KEY_RAW: ${{ secrets.SNOWFLAKE_PRIVATE_KEY_RAW }}
        run: task test-snow-connection
```

In order to allow the GitHub Actions workflow to access the Snowflake database, we need to create a network policy that allows the GitHub Actions workflow to access the Snowflake database. Here we are using the `GITHUBACTIONS_GLOBAL` network rule. This rule is a global rule that allows all [GitHub-hosted runners](https://docs.github.com/en/actions/concepts/runners/github-hosted-runners) to access the Snowflake database. In an enterprise environment, you can further harden your security by creating self-hosted runners and creating a network policy that allows only the self-hosted runners' IP address to access the Snowflake database.

```sql
CREATE OR REPLACE NETWORK POLICY github_actions_ingress ALLOWED_NETWORK_RULE_LIST = (
  'SNOWFLAKE.NETWORK_SECURITY.GITHUBACTIONS_GLOBAL'
);
```

For information on the network rules available, you can run the following queries:

Show all network rules:

```sql
SHOW NETWORK RULES IN SNOWFLAKE.NETWORK_SECURITY;
```

Show the details of the `GITHUBACTIONS_GLOBAL` network rule:

```sql
SELECT *
  FROM SNOWFLAKE.ACCOUNT_USAGE.NETWORK_RULES
  WHERE DATABASE = 'SNOWFLAKE' 
    AND SCHEMA = 'NETWORK_SECURITY'
    AND NAME = 'GITHUBACTIONS_GLOBAL';
```

## GitHub Actions Workflow

The GitHub Actions workflow is defined in the `.github/workflows/blank.yml` file. This workflow is a basic workflow that will test the Snowflake CLI connection.
