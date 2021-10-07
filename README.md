# Terraform Credentials from HashiCorp Vault

`terraform-credentials-vault` is a Terraform "credentials helper" plugin that
allows providing credentials for
[Terraform-native services](https://www.terraform.io/docs/internals/remote-service-discovery.html)
(private module registries, Terraform Cloud, etc) via environment variables.

It is based on [apparentlymart/terraform-credentials-env](https://github.com/apparentlymart/terraform-credentials-env)

To use it,
[download a release archive](https://github.com/oulman/terraform-credentials-vault/releases)
and extract it into the `~/.terraform.d/plugins` directory where Terraform
looks for credentials helper plugins. (The filename of the file inside the
archive is important for Terraform to discover it correctly, so don't rename
it.)

Terraform will take the newest version of the plugin it finds in the plugin
search directory, so if you are switching between versions you may prefer to
remove existing installed versions in order to ensure Terraform selects the
desired version.

Once you've installed the plugin, enable it by adding the following block
to your
[Terraform CLI configuration](https://www.terraform.io/docs/commands/cli-config.html):

```hcl
credentials_helper "vault" {
    args = ["--vault-path=/secret/data/gitlab/terraform_registry"]
}
```

With this helper installed and enabled, you can set credentials for specific
hostnames in the environment for your shell so that they will be inherited
by `terraform` and then in turn by `terraform-credentials-vault`.

The helper will use your existing Vault environment settings like `VAULT_ADDR` and `~/.vault-token` or `VAULT_TOKEN` for your
token.

The Vault path must use the kv2 secrets engine and most contain a secret matching hostname
with a field of token. Example: for a --vault-path of secrets/data/terraform_registry you
and a hostname of gitlab.corp.com `terraform-credentials-vault` will search at `secrets/data/terraform_registry/gitlab.com`
and use the value in the token field.

Terraform will execute the configured credentials helper plugin whenever it
needs to make a request to a Terraform-native service whose credentials aren't
directly configured in the CLI configuration using `credentials` blocks.
`credentials` blocks override credentials helpers though, so if you have any
existing `credentials` block for the hostname you wish to configure you will
need to remove that block first.
