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
* Static credentials should be used here so they can be controlled outside of Vault. However, these static credentials should still be stored within Vault using the KV secrets engine so they are not stored somewhere in plaintext.
* You can indeed create and update Vault policies within the UI.
* Vault Enterprise offers additional features that allow HA nodes to service read-only requests on the local standby node. Read-only requests are requests that do not modify Vault's storage. This feature is called Performance Standby nodes.
* Beyond Cubbyhole, the KV secrets engine is the ONLY secrets engine that will store static data in Vault for future retrieval. All other secrets engines either generate or encrypt data.
* The Transit and Transform secrets engine can encrypt/decrypt data can encrypt and decrypt data for Vault clients.
* To invoke an API on a specific namespace, you can pass the target namespace in the X-Vault-Namespace header or make the namespace as a part of the API endpoint.
* When authenticating to Vault using the API, Vault will return a token. It is up to the operator or automation to extract the token from the response and submit it as part of subsequent requests
* Batch tokens cannot create child tokens, nor are they renewable.
* Service token is the general token that most people talk about when referring to a token in Vault.
Batch token is an encrypted binary large object (blobs) that carries just enough information for authentication.
Periodic service tokens have a TTL, but no max TTL.
Orphan tokens are not children of their parent; therefore, do not expire when their parent does.
* Performance replication clusters should be used in an active/active scenario to ensure applications in both data centers can easily access Vault secrets.
* n order versions of Vault (pre-Vault 1.10), batch and service tokens are prepended with b. and s., respectively. In newer versions of Vault (Vault 1.10+), batch tokens are prepended with hvb. Batch tokens are also longer (meaning byte size) than service tokens because batch tokens are encrypted by the Vault's barrier.
* Vault Agent is a client daemon that provides the following features:
  - Auto-Auth - Automatically authenticate to Vault and manage the token renewal process for locally-retrieved dynamic secrets.
  - Caching - Allows client-side caching of responses containing newly created tokens and responses containing leased secrets generated off of these newly created tokens.
  - Templating - Allows rendering of user supplied templates by Vault Agent, using the token generated by the Auto-Auth step.
* The vault secrets command has several subcommands to use when working with secrets engines, including:
  - disable - Disable a secrets engine
  - enable - Enable a secrets engine
  - list - List enabled secrets engines
  - move - Move a secrets engine to a new path
  - tune - Tune a secrets engine configuration.
* Active Directory is enabled by the LDAP auth method, and not an Active Directory auth method.
* Vault's storage backend is inherently untrusted, therefore Vault manages the encryption and decryption of the data when reading or writing to disk.
* Vault Agent Caching allows client-side caching of responses containing newly created tokens and responses containing leased secrets generated off of these newly created tokens. The renewals of the cached tokens and leases are also managed by the agent.
* Vault supports two different types of replication, performance and disaster recovery (DR). Performance clusters create and maintain their own tokens. These tokens are NOT replicated to other clusters. DR clusters are essentially a warm-standby and do replicate tokens from the primary cluster.
* If you're not running dev mode, Vault no longer enables a default KV secrets engine like it used to.
* Internal and external groups can be created in Vault. Internal groups are created in the identity store and map to other groups or entities. Internal groups are created manually as needed. External groups are usually associated with an auth method, such as LDAP or OIDC.
* When creating the key, the exportable flag must be set as true. By default, it is false. This enables the keys to be exportable.
* You can use the rewrap feature of the transit secrets engine to rewrap the data with the latest version of the key. This process does not reveal the plaintext data.
* When no specific TTL is provided, a generated token will inherit the default TTL which is 2764800 seconds (32 days). The same for the maximum TTL.
* AppRole is an auth method that is better suited for machine-to-machine authentication.
* Performing a rekey operation using the vault operator rekey command creates new unseal/recovery keys as well as a new master key.
* If a lease has been created in Vault, it has an associated TTL in which it will expire and be revoked. If the lease needs to be extended for some reason, you can use the command **vault lease renew <lease_id>** to extend the TTL of the lease so it will not expire at its original TTL and will be extended by the time specified in seconds from the current time the lease renewal was issued.
* Out of the options provided, only two of them are valid headers to use to set the token in a Vault API request.
X-Vault-Token: <token>
Authorization: Bearer <token>
* When creating a policy, only the following capabilities are available in Vault:
  - Create
  - Update
  - Read
  - Delete
  - List
  - Sudo
  - Deny
* The big differences between the two types of replication include:
  * DR replication will replicate all tokens and leases from the primary cluster to the secondary. This means tokens that were valid for the primary cluster are valid for the secondary cluster when it is promoted. However, a DR replication cluster does NOT respond to clients unless it is promoted to a primary.
  * Performance replication can respond to client requests, but it handles its own tokens and leases. Any tokens or leases that are created on the primary cluster are NOT replicated to the secondary servers. Therefore if you failover to the secondary cluster, applications would need to re-authenticate because the existing tokens would not be valid on the secondary cluster.
* Vault does not trust the storage backend where it stores its data, therefore all data that is written to the storage backend passing through the cryptographic barrier and is encrypted.
* When tokens are created, a token accessor is also created and returned to the user/requested. This accessor is a value that acts as a reference to a token and can only be used to perform limited actions in Vault. These actions include 1) looking up information about the token, 2) renewing the token, 3) lookup the capabilities on a path, and 4) revoking the token.
* When you use the Vault API to authenticate, the Vault API response will include a client_token that is tied to a specific policy. Once you receive that response, it is up to the user (or application) to parse that response and retrieve the token. Once the token is retrieved, a second API request needs to be sent to Vault to request the new PKI certificate.
* Vault supports a single storage backend to store its encrypted data.
* Tokens can be used on any platform since they are created and managed by Vault itself.
* The PKI secrets engine generates dynamic X.509 certificates.
* There are lots of benefits of using Integrated Storage as a storage backend for Vault. Introduced in Vault 1.4, Integrated Storage is a built-in solution that provides a highly available, durable storage backend without relying on any external systems. Integrated Storage uses the same underlying consensus protocol (RAFT) as Consul to handle cluster leadership and log management. All Vault data is stored locally on each node, and replicated to all other nodes in the cluster for high availability. It also reduces complexity since all configuration is done within Vault. No external systems to provision alongside Vault.
* Legacy applications often suffer from the ability to integrate with modern platforms such as Vault. To assist with this, you can use the Vault Agent to authenticate and manage a Vault token automatically.
* In fact, there are only three ways to create root tokens:
  * The initial root token generated at vault operator init -- this token has no expiration
  * By using another root token; a root token with an expiration cannot create a root token that never expires
  * By using vault operator generate-root (example) with the permission of a quorum of unseal/recovery key holders
* When you need Vault to store credentials that were created by a third-party platform, such as Active Directory or API keys from a partner website, you need to use the KV secrets engine. The KV secrets engine is the ONLY secrets engine that can actually store credentials in Vault.
* Not all storage backends are created equal. Some support high availability, some do not.
* Vault provides an easy way of re-wrapping encrypted data when a key is rotated. Using the rewrap API endpoint, a non-privileged Vault entity can send data encrypted with an older version of the key to have it re-encrypted with the latest version.
* With every dynamic secret and service type authentication token, Vault creates a lease. A lease is metadata containing information such as time duration, renewability, and more.
* When you are retrieving data on a KV Version 2 secrets engine, you must include the data/ prefix, since a KV V2 secrets engine splits data and metadata into different prefixes.
* VAULT_ADDR is the environment variable that is used to specify the address of the Vault server expressed as a URL and port.
* When using the Vault CLI, the default output is TABLE, but you can use the -format flag to specify another output format, such as a JSON or YAML.
* The policy must also include the sudo capability in order to permit the user to rotate the encryption key for Vault. Alternatively, you can use a root token to perform this action.
* The Transit secrets engine is used to encrypt data in transit. It does NOT store the data locally. It simply encrypts the data and returns the ciphertext to the requester.
* To view information about a token, the command **vault token lookup** can be used on the CLI.
* When the vault initialize command is fired, the vault will give both unseal keys and root token. Root Token is not advised to be kept after the entire configuration becomes complete.
* Vault can be unsealed by running the command “vault operator unseal”.
* Vault can be initialized by running the command “vault operator init”.
* Vault has two replications – Performance & DR.
* To create a token: **curl \ --header "X-Vault-Token: ..." \ --request POST \ --data @payload.json \ http://127.0.0.1:8200/v1/auth/token/create**
* Github is one of the auth methods available, and it can be enabled via **vault auth enabled Github**.
* To change TTL to 1 hour use **vault lease renew -increment=3600 my-lease-id**.
* **VAULT_TOKEN** is the environment variable used for the Vault authentication token.
* Vault uses port 8200 for the API.
* Create, Read, Update, Delete, List, Sudo, deny are the available capabilities for the policies.
* **vault token create -no-default-policy** to create a token without a default policy.
* Batch Token can’t be used to create tokens.
* Root tokens may have a TTL associated, but the TTL may also be 0, indicating a token that never expires.
* Wrapping Token is used for securing the secret zero when the App Role auth method is used.
* **vault write transit/encrypt/<key_ring_name>** is the command to encrypt secrets.
* **transit/decrypt/** is the endpoint to decrypt the ciphertext.
* **vault secrets enable transit** is the command to enable the transit engine.
* We are allowed to customize the mount points for each Auth Method that is enabled.
* When an auth method is disabled, all users authenticated via that method are automatically logged out.
* Read Capability should be defined in the policy by mentioning only Read in the syntax.
* Default policies cannot be removed directly. The default policy is a built-in Vault policy that cannot be removed. By default, it is attached to all tokens, but may be explicitly excluded at token creation time by supporting authentication methods.
* The AppRole auth method allows machines or apps to authenticate with Vault-defined roles.
* **auth/token/revoke-orphan** is the endpoint to revoke the orphan.
* Transit Secret engine handles cryptographic functions on data in transit.
* PKI secrets engine generates dynamic X.509 certificates.
* By default, any token generated from the Parent inherits the parent policies.
* Root Token generated during initialization has no expiry by default. Root tokens cannot be used to generate another Root token with expiry.
* Correct syntax to read the secrets in KV using API is **/secret/data/:path?version=:version-number**.
* **curl \ --header "X-Vault-Token: ..." \ http://127.0.0.1:8200/v1/cubbyhole/my-secret**  is the right curl command to read secrets with the cubbyhole secrets engine.
* The lease & lease_max are the right parameters for defining the leases.
* KMIP engine requires Vault Enterprise with the Advanced Data Protection Module.
* **http://127.0.0.1:8200/ui** is the default location of the vault UI.
* The cubbyhole secrets engine is enabled by default. It cannot be disabled, moved, or enabled multiple times.
* By default, Vault enables Key/Value version2 secrets engine (KV-v2) at the path secret/ when running in dev mode.
* During the encryption of Ciphertext, the v1 in the command denotes to the version of encryption key used.
* **vault write transit/keys/orders/config min_decryption_version=4** enforces the use of a specific version of the encryption key during decryption operation.
* We can have a mount that is prefixed with an existing mount when creating secret engines because the mount prefix is unique in nature.
* You can move the secret to a different path using the move parameter.
* **allowed_parameters** is used to whitelist the keys and values through Vault Policy.
* The token can be generated with the limitation enforced on the number of uses
* Vault operator can create external and internal groups.
* The **vault login** is used for logging in to vault CLI.
* Vault secrets engines are used to store, generate, or encrypt data.
* When Vault is sealed, the only two options available are, viewing the vault status and unsealing the Vault.
* **vault lease revoke -prefix <path>** will remove all secrets at a specific path.
* Same auth methods can’t be enabled at the same paths.
* The path **/auth/token/lookup-self** is used to get information about the current client token
* The parameter **secret_id_num_uses** is used to set the maximum number of times that any particular SecretID can be used to fetch a token from the AppRole, after which the SecretID will expire.
* The maximum allowed request size is 32MB.
* Vault won’t store any data that has been encrypted with the help of a transit secret engine.
* The command “vault secret move” is used to move an existing secrets engine to a new path. During this process, the leases associated with the secret engine are revoked, but the configuration associated with the engine is preserved.
* vault secrets enable -max-lease-ttl=45m –path=mongodb database
* The token accessor can only be used to perform limited actions which are as follows:
  * Look up a token's properties (not including the actual token ID)
  * Look up a token's capabilities on a path
  * Renew the token
  * Revoke the token
* The root tokens can be created in any one of the following ways:
  * The initial root token is generated at vault operator init time -- this token has no expiration.
  * By using another root token, a root token with an expiration cannot create a root token that never expires.
  * By using vault operator generate-root with the permission of a quorum of unseal key holders.
  * By using the endpoint /sys/generate-root.
* The environment variable **VAULT_ADDR** is used to set the address of the Vault server in the format of URL:PORT
* Writing to a key in the cubbyhole secrets engine will completely replace the old value, and it won’t persist the old value.
* The TOTP secrets engine can act as a TOTP code generator. It can replace traditional TOTP generators like Google Authenticator.
* The port 8200 is used for the purpose of cluster bootstrapping in the Vault servers. Also, the port is used for Vault API communication.
* The following storage backends are supported by Hashicorp:
  * Consul
  * Filesystem
  * In-Memory
  * Integrated Storage (Raft)
* Vault Agent comes by default with the Vault binary file.
* The cubbyhole secrets engine is used to achieve the cubbyhole response wrapping where the initial token is stored.
* Vault does not store any of the unseal key shards in any persistent storage.
