# hashicorp-certified-vault-associate

* Namespaces are isolated environments that functionally exist as "Vaults within a Vault." They have separate login paths and support creating and managing data isolated to their namespace.
* Telemetry information can be streamed directly from Vault to a range of metrics aggregation solutions.
* **tokens** are the default auth method in Vault and cannot be disabled.
* The revocation can happen manually via the API, via the ```vault lease revoke``` cli command, the user interface (UI) under the Access tab, or automatically by Vault.
* When encrypting data using Vault, the ciphertext is always prepended with the version of the encryption key that was used. In the case the version is v2 it means that the encryption key was rotated at least one time. Any data that was encrypted with the original key would have been prepended with vault:v1.
* The Key/Value Backend, which stores arbitrary secrets does not issue leases although it will sometimes return a lease duration.
* Consul ACLs should always be enabled when using Consul as a storage backend. This policy allows Vault to communicate to required services hosted on Consul.
* Vault does NOT store any data encrypted via the transit/encrypt endpoint. The output you received is the ciphertext.
* Certificates are not a valid unseal mechanism for HashiCorp Vault.
* The Vault transit secrets engine does NOT store any data, it only encrypts/decrypts data and returns the value upon request.
* Once you create a KV v1 secrets engine and place data in it, there is a way to modify the mount to include the features of a KV v2 secrets engine.
* 
