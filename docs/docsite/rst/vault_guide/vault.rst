.. _vault:

*************
Ansible Vault
*************

Ansible Vault encrypts files and strings so you can protect sensitive content such as passwords or keys rather than leaving it visible as plaintext in playbooks or roles.
These vault strings can be assigned to a variable and Ansible will decrypt them when needed, files are decrypted when read.
To use Ansible Vault you need one or more secrets to encrypt and decrypt content.
You can use several methods for providing your vault secrets, via a prompt, in a restricted file or even using a script to access a 3rd party storage tool.
Use the passwords with the :ref:`ansible-vault` command-line tool to create and view encrypted strings or files, encrypt existing files, or edit, re-key, or decrypt as needed.

.. warning::
    * Encryption with Ansible Vault ONLY protects 'data at rest'.
      Once the content is decrypted ('data in use'), play and plugin authors are responsible for avoiding any secret disclosure,
      see :ref:`no_log <keep_secret_data>` for details on hiding output and :ref:`vault_securing_editor` for security considerations
      on editors you use with Ansible Vault.

You can use encrypted variables and files in ad hoc commands and playbooks by supplying the secrets you used to encrypt them.
You use configuration (:ref:`DEFAULT_VAULT_PASSWORD_FILE`) to specify the location of a secrets file/script or prompt for the secrets.
