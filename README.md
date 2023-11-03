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
* The plus sign (+) can be used to denote a path segment and can be used in the middle of a path. The splat (*) can be used as a wildcard but can only be used at the very end of a path.
* The userpass auth method uses a local database that cannot interact with any services outside of the Vault instance.
* When creating a dynamic secret, Vault always returns a lease_id. This lease_id can be used to do a **vault lease renew** or a **vault lease revoke** command to manage the lease of a secret.
* **vault lease revoke -force -prefix <lease_path>** can be run to make Vault remove the secret.
* The transit secrets engine handles cryptographic functions on data in-transit. Vault doesn't store the data sent to the secrets engine. It can also be viewed as "cryptography as a service" or "encryption as a service". The transit secrets engine can also sign and verify data; generate hashes and HMACs of data; and act as a source of random bytes.
* Vault creates two default policies, root and default. The root policy cannot be deleted or modified. The default policy is attached to all tokens, by default, however, this action can be modified if needed.
* VAULT_TOKEN defines that Vault authentication token. Conceptually similar to a session token on a website, the VAULT_TOKEN environment variable holds the contents of the token.
* Non-root tokens are associated with a TTL, which determines for how long a token is valid. Root tokens are not associated with a TTL, and therefore, do not expire.
* In order to enable auth methods, the command should be **vault auth <enable/disable>** followed by the name of the auth method.
* The Vault configuration file supports either JSON or HCL, which is HashiCorp Configuration Language.
* The KVv2 store uses a prefixed API, which is different from the version 1 API. Writing and reading versions are prefixed with the data/ path. I
* A lease must be renewed before it has expired. Once it has expired, it is permanently revoked and a new secret must be requested.
* After authenticating, a client is issued a service token which is associated with a policy. That token is used to make all subsequent requests to Vault.
* Key versions that are earlier than a key's specified **min_decryption_version** gets archived, and the rest of the key versions belong to the working set. In an emergency, the **min_decryption_version** can be moved back to allow for legitimate decryption.
* To write a policy, use the vault policy write command.
* **allowed_parameters** can be used to permit a list of keys and values that are permitted on the given path. Setting a parameter with a value of the empty list allows the parameter to contain any value.
* After initializing, Vault provides the user the root token, which is the only way to log in to Vault in order to configure additional auth methods.
* Vault Transit secrets engine does not support an action to update since Vault does not store any data. You can, however, rewrap data when the key has been rotated to ensure data is encrypted with the latest version.
* Batch tokens are lightweight and scalable and include just enough information to used with Vault. They are generally used for ephemeral, high-performance workloads, such as encrypting data.
* Since the encryption key is stored in memory, Vault nodes do not share or replicate the encryption key to other nodes. Therefore, each node needs to individually unseal itself upon Vault initialization or anytime the Vault service is restarted on that node.
* Auto unseal is used to automatically unseal Vault but using an HSM or cloud HSM service.
* After authenticating, the UI and CLI automatically assume the token for all subsequent requests.
* In order to renew a token, a user can issue a **vault token renew** command to extend the TTL. The token can also be renewed using the API.
* To login with a token, you can use **vault login hvs.hxDIPd8RPVtxu4AzSGS1lArP** or even **vault login -method=token hvs.hxDIPd8RPVtxu4AzSGS1lArP** if you like typing more.
* If the UI doesn't support a required configuration, the user can potentially use the Vault CLI Browser to execute simple write, read, delete, and list commands.
* Vault Agent is a client daemon that provides auto-auth, caching, and templating.
* The **vault lease revoke** command is used to revoke leases. Using the **-prefix** flag allows you to revoke entire trees of secrets.
* Vault has many secrets engines that can generate dynamic credentials, including AWS, Azure, and database secrets engines. The key/value secret engine is used to store data, the transit secret engine is used to encrypt data.
* By default, Vault starts the Vault service on port 8200. Cluster-to-cluster communication and replication is done over 8201.
* Storage backends, cluster names, and seal types are just a few of the configurations done via the configuration file. The others are configured within Vault itself.
* When writing a policy in Vault, permissions which can be applied to paths include create, read, update, delete, list, deny, and sudo
* In order to log in to Vault via the CLI, you would issue the command 'vault login' and then the type of login.
* Vault supports AWS, Azure, Google Cloud, and Alibaba Cloud out of the box for secrets engines.
* The process of unsealing is used to reconstruct the master key, which is used to decrypt the encryption key which can unencrypt the data on the storage backend.
* The **vault operator step-down** forces the Vault server at the given address to step down from active duty. While the affected node will have a delay before attempting to acquire the leader lock again, if no other Vault nodes acquire the lock beforehand, it is possible for the same node to re-acquire the lock and become active again.
* The **lease revoke** command revokes the lease on a secret, invalidating the underlying secret. To revoke a lease, you can specify the path and lease ID attached to the creds. Once executed, the credentials on the backend platform are immediately revoked and no longer valid.
* A token accessor is created alongside of each token, and the accessor can be used to perform limited actions against the token, including looking up the token's properties, renewing the token, and even revoking the token.
* To initialize Vault, use the command 'vault operator init'.
* Although Vault has many benefits, Vault does not provide service discovery for micro-segmentation.
* The UI is enabled in the Vault configuration file, not the CLI.
* All backend system functions live in the /sys backend. Policies should take /sys into account when users need to administer Vault configurations.
* When having a token be revoked would be problematic, root or sudo users have the ability to generate periodic tokens. Periodic tokens have a TTL, but no max TTL. Periodic tokens may live for an infinite amount of time, so long as they are renewed within their TTL.
* For day-to-day operations, the root token should be deleted after configuring other auth methods which will be used by admins and Vault clients.
* While Vault is sealed, the only two options available are viewing the vault status (vault status) and unsealing Vault (vault operator unseal).
* When a parent token is revoked, all of its child tokens -- and all of their leases -- are revoked as well.
* To list all enabled secrets engines with detailed output, use the command **vault secrets list -detailed**.
* Dynamic secrets are generated when they are accessed. Dynamic secrets do not exist until they are read, so there is no risk of someone stealing them or another client using the same secrets. Because Vault has built-in revocation mechanisms, dynamic secrets can be revoked immediately after use, minimizing the amount of time the secret existed.
* Orphan tokens are not children of their parent; therefore, orphan tokens do not expire when their parent does.
* The default and root policy cannot be deleted. You don't have to use them, but you can't delete them. For the default policy, you can instruct Vault to not assign new tokens the default policy by tuning the Vault configuration by issuing the following command: **vault token create -no-default-policy**.
* Batch tokens cannot be renewed by Vault, but service tokens can be renewed up to the Max TTL of the token.
* Unsealing is the process of obtaining the plaintext master key necessary to read the decryption key to decrypt the data, allowing access to the Vault.
Decrypting the Vault data is a result of unsealing Vault, but the process of unsealing Vault does not directly decrypt the Vault data.
Vault assumes the value of **https://127.0.0.1:8200** when you make requests to Vault.
* The token for authentication is set directly as a header for the HTTP API. The header should be either **X-Vault-Token: <token>** or **Authorization: Bearer <token>**.
* 
