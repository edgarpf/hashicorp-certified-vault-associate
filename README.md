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
* Replication relies on having a shared keyring between primary and secondaries and also relies on having a shared understanding of the data store state. As a result, when replication is enabled, all of the secondary's existing storage will be wiped.
* To grant access to generate database credentials, the policy would grant read access on the appropriate path.
* Tokens can be renewed with the vault token renew command. If not renewing the locally authenticated token, you need to signify what token should be renewed by either the **token_id** or the **token accessor**.
* The system max TTL, which is 768 hours, or 32 days.
* The AD secrets engine rotates AD passwords dynamically. It does not, however, dynamically generate the AD account. The AD account must exist prior to configuring it in Vault.
* Before Vault can be used, it must be initialized and unsealed.
* Plugins are the components in Vault that can be implemented separately from Vault's built-in backends.
* Using replication requires a storage backend that supports transactional updates, such as Consul. This allows multiple key/value updates to be performed atomically. Replication uses this to maintain a Write-Ahead-Log (WAL) of all updates so that the key update happens atomically with the WAL entry creation.
* Dev server mode stores its data in memory, therefore if the Vault service is shut down, any data stored will be lost. Additionally, dev server mode does not use TLS, and all data is sent in cleartext.
* Vault is sending the request over HTTPS, but Vault is responding using HTTP since TLS is disabled. In this case, you should set the VAULT_ADDR environment variable to http://127.0.0.1:8200. This is true if you're running Vault Dev server as well.
* The Cubbyhole secret engine is a default secrets engine that is enabled for each Vault token.
* Batch tokens are encrypted blobs that carry enough information for them to be used for Vault actions, but they require no storage on disk to track them. As a result, they are extremely lightweight and scalable but lack most of the flexibility and features of service tokens.
* The /sys/leader endpoint is used to check the high availability status and current leader of Vault.
* **https://vault.krausen.com:8200/v1/sys/tools/random/164*** endpoint returns high-quality random bytes of the specified length.
* Vault has many ports, however, replication occurs on port tcp/8201.
* The TTL defines when the token will expire. If the token reaches its TTL, it will be immediately revoked by Vault. The Max TTL defines the maximum timeframe for which the token can be renewed. Once the max TTL is reached, the token cannot be renewed any longer and will be revoked.
* The kv delete command deletes the data for the provided path in the key/value secrets engine. If using K/V Version 2, its versioned data will not be fully removed but marked as deleted and will no longer be returned in normal get requests.
* The kv destroy command permanently removes the specified versions' data from the key/value secrets engine. If no key exists at the path, no action is taken.
* The kv metadata delete command deletes all versions and metadata for the provided key.
* All plaintext data must be base64-encoded. The reason for this requirement is that Vault does not require that the plaintext is "text".
* The command vault **kv delete** only deletes the current version of the secret, and by delete, I mean it does a "soft" delete. All the remaining versions (if any) will still exist - so will the metadata.
* The following policy will give access to anything in Vault. 
  ```
  path "*" {
    capabilities = ["create", "update", "read", "list", "delete", "sudo"]
  }
  ```
* The PKI secrets engine generates dynamic X.509 certificates. With this secrets engine, services can get certificates without going through the usual manual process of generating a private key and CSR, submitting to a CA, and waiting for a verification and signing process to complete. Vault's built-in authentication and authorization mechanisms provide the verification functionality.
By keeping TTLs relatively short, revocations are less likely to be needed, keeping CRLs short and helping the secrets engine scale to large workloads. This, in turn, allows each instance of a running application to have a unique certificate, eliminating sharing and the accompanying pain of revocation and rollover.
In addition, by allowing revocation to mostly be forgone, this secrets engine allows for ephemeral certificates. Certificates can be fetched and stored in memory upon application startup and discarded upon shutdown, without ever being written to disk.
* Each auth method serves a different purpose, and some auth methods are better suited for machine authentication rather than used by human users. Examples of machine auth methods include AppRole, Cloud-based auth methods, tokens, TLS, Kubernetes, and Radius. Examples of human auth methods include Okta, LDAP, GitHub, OIDC, and userpass.
* 
