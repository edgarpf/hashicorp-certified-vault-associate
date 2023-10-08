# hashicorp-certified-vault-associate

* Namespaces are isolated environments that functionally exist as "Vaults within a Vault." They have separate login paths and support creating and managing data isolated to their namespace.
* Telemetry information can be streamed directly from Vault to a range of metrics aggregation solutions.
* **tokens** are the default auth method in Vault and cannot be disabled.
* The revocation can happen manually via the API, via the ```vault lease revoke``` cli command, the user interface (UI) under the Access tab, or automatically by Vault.
* Vault includes a feature called response wrapping. With it you can encrypt sensitive data.
* The command format for enabling Vault features is ```vault <feature> <enable/disable> <name>```
* When encrypting data using Vault, the ciphertext is always prepended with the version of the encryption key that was used. In the case the version is v2 it means that the encryption key was rotated at least one time. Any data that was encrypted with the original key would have been prepended with vault:v1.
* The Key/Value Backend, which stores arbitrary secrets does not issue leases although it will sometimes return a lease duration.
* Consul ACLs should always be enabled when using Consul as a storage backend. This policy allows Vault to communicate to required services hosted on Consul.
* Vault does NOT store any data encrypted via the transit/encrypt endpoint. The output you received is the ciphertext.
* Certificates are not a valid unseal mechanism for HashiCorp Vault.
* The Vault transit secrets engine does NOT store any data, it only encrypts/decrypts data and returns the value upon request.
* Once you create a KV v1 secrets engine and place data in it, there is a way to modify the mount to include the features of a KV v2 secrets engine.
* The Transit engine supports the versioning of keys. Key versions that are earlier than a key's specified ```min_decryption_version``` get archived, and the rest of the key versions belong to the working set. This is a performance consideration to keep key loading fast, as well as a security consideration: by disallowing decryption of old versions of keys, found ciphertext corresponding to obsolete (but sensitive) data can not be decrypted by most users, but in an emergency, the ```min_decryption_version``` can be moved back to allow for legitimate decryption.
* The TOTP secrets engine can act as a TOTP code generator. In this mode, it can replace traditional TOTP generators like Google Authenticator.
* When a parent token is revoked, all of its child tokens -- and all of their leases -- are revoked as well. This ensures that a user cannot escape revocation by simply generating a never-ending tree of child tokens.
* For replication (port 8201), Vault creates a mutual TLS connection between nodes using self-generated certs/keys (this is different than the TLS you configure in the listener, which is specific to client requests)... server-to-server ALWAYS uses this mutual TLS, even if you have TLS disabled on the listener. If traffic between cluster nodes is forced through the load balancer, the TLS termination will break replication.
* ```vault write transit/rewrap/ecommerce ciphertext=<old data>``` can be used to easily re-encrypt the original data with the new version of the key.
* When you send data to Vault for encryption, it must be in the form of base64-encoded plaintext for safe transport.
* The plugin directory is a configuration option of Vault, and can be specified in the configuration file. This setting specifies a directory in which all plugin binaries must live; this value cannot be a symbolic link. A plugin can not be added to Vault unless it exists in the plugin directory. There is no default for this configuration option, and if it is not set plugins can not be added to Vault.
* When enabled, auth methods are similar to secrets engines: they are mounted within the Vault mount table and can be accessed and configured using the standard read/write API. All auth methods are mounted underneath the ```auth/``` prefix. By default, auth methods are mounted to auth/<type>. For example, if you enable "github", then you can interact with it at ```auth/github```.
* In order to browse through the UI, the user needs **list** permissions on all paths leading up to the path in which they are allowed to read.
