.. _vault_using_encrypted_content:
.. _playbooks_vault:
.. _providing_vault_passwords:

Using encrypted variables and files
===================================

When you run a task or playbook that uses encrypted variables or files, you must provide the passwords to decrypt the variables or files. You can do this at the command line or by setting a default password source in a config option or an environment variable.

Passing a single password
-------------------------

If all the encrypted variables and files in your task or playbook need to use a single password, you can use the :option:`--ask-vault-pass <ansible-playbook --ask-vault-pass>` or :option:`--vault-password-file <ansible-playbook --vault-password-file>` cli options.

To prompt for the password:

.. code-block:: bash

    ansible-playbook --ask-vault-pass site.yml

To retrieve the password from the :file:`/path/to/my/vault-password-file` file:

.. code-block:: bash

    ansible-playbook --vault-password-file /path/to/my/vault-password-file site.yml

To get the password from the vault password client script :file:`my-vault-password-client.py`:

.. code-block:: bash

    ansible-playbook --vault-password-file my-vault-password-client.py


.. _specifying_vault_ids:

Passing vault IDs
-----------------

You can also use the :option:`--vault-id <ansible-playbook --vault-id>` option to pass a single password with its vault label. This approach is clearer when multiple vaults are used within a single inventory.

To prompt for the password for the 'dev' vault ID:

.. code-block:: bash

    ansible-playbook --vault-id dev@prompt site.yml

To retrieve the password for the 'dev' vault ID from the :file:`dev-password` file:

.. code-block:: bash

    ansible-playbook --vault-id dev@dev-password site.yml

To get the password for the 'dev' vault ID from the vault password client script :file:`my-vault-password-client.py`:

.. code-block:: bash

    ansible-playbook --vault-id dev@my-vault-password-client.py

Passing multiple vault passwords
--------------------------------

If your task or playbook requires multiple encrypted variables or files that you encrypted with different vault IDs, you must use the :option:`--vault-id <ansible-playbook --vault-id>` option, passing multiple ``--vault-id`` options to specify the vault IDs ('dev', 'prod', 'cloud', 'db') and sources for the passwords (prompt, file, script). For example, to use a 'dev' password read from a file and to be prompted for the 'prod' password:

.. code-block:: bash

    ansible-playbook --vault-id dev@dev-password --vault-id prod@prompt site.yml

By default, the vault ID labels (dev, prod and so on) are only hints. Ansible attempts to decrypt vault content with each password. The password with the same label as the encrypted data will be tried first, after that, each vault secret will be tried in the order they were provided on the command line.

Where the encrypted data has no label, or the label does not match any of the provided labels, the passwords will be tried in the order they are specified. In the example above, the 'dev' password will be tried first, then the 'prod' password for cases where Ansible doesn't know which vault ID is used to encrypt something.

Using ``--vault-id`` without a vault ID
---------------------------------------

The :option:`--vault-id <ansible-playbook --vault-id>` option can also be used without specifying a vault-id. This behavior is equivalent to :option:`--ask-vault-pass <ansible-playbook --ask-vault-pass>` or :option:`--vault-password-file <ansible-playbook --vault-password-file>` so is rarely used.

For example, to use a password file :file:`dev-password`:

.. code-block:: bash

    ansible-playbook --vault-id dev-password site.yml

To prompt for the password:

.. code-block:: bash

    ansible-playbook --vault-id @prompt site.yml

To get the password from an executable script :file:`my-vault-password-client.py`:

.. code-block:: bash

    ansible-playbook --vault-id my-vault-password-client.py

Configuring defaults for using encrypted content
================================================

Setting a default vault ID
--------------------------

If you use one vault ID more frequently than any other, you can set the config option :ref:`DEFAULT_VAULT_IDENTITY_LIST` to specify a default vault ID and password source. Ansible will use the default vault ID and source any time you do not specify :option:`--vault-id <ansible-playbook --vault-id>`. You can set multiple values for this option. Setting multiple values is equivalent to passing multiple :option:`--vault-id <ansible-playbook --vault-id>` cli options.

Setting a default password source
---------------------------------

If you don't want to provide the password file on the command line or if you use one vault password file more frequently than any other, you can set the :ref:`DEFAULT_VAULT_PASSWORD_FILE` config option or the :envvar:`ANSIBLE_VAULT_PASSWORD_FILE` environment variable to specify a default file to use. For example, if you set ``ANSIBLE_VAULT_PASSWORD_FILE=~/.vault_pass.txt``, Ansible will automatically search for the password in that file. This is useful if, for example, you use Ansible from a continuous integration system such as Jenkins.

The file that you reference can be either a file containing the password (in plain text), or it can be a script (with executable permissions set) that returns the password.

When is the encrypted data made visible?
========================================

In general, the content you encrypt with Ansible Vault remains encrypted until needed. This is different depending on how you use the vaults.

    * For file vaults and some actions in which you pass them as the ``src`` argument to the :ref:`copy <copy_module>`, :ref:`template <template_module>`, :ref:`unarchive <unarchive_module>`, :ref:`script <script_module>` or :ref:`assemble <assemble_module>` module, the file will be decrypted on the controller and pushed in the clear to the target host (assuming you supply the correct vault password when you run the play). This is designed so can encrypt a configuration file or template to avoid sharing the details of your configuration, but when you copy that configuration to servers in your environment, you want it to be decrypted so local users and processes can access it.
     This is the default behaviour but the user can change that via the ``decrypt`` option. Disabling this  will result in the actions copying a vaulted/encrypted copy to the target.

    * Vaulted variable files are decrypted on load as Ansible needs to know the variable names available, which are also encrypted when it is a fully vaulted file. This is true no matter how they are loaded, as extra vars, ``vars_files``, ``include_vars`` or a vars plugin.

    * Inline vaulted variables are decrypt when consumed, this is normally done at the task level when resolving the value for a playbook keyword or when assigned an action's options.


.. note::

    * Like templating, all decryption happens on the controller, this avoids having to disclose vault passwords to the targets.

.. _vault_format:

The Vault Format
================

Ansible Vault creates UTF-8 encoded txt. The format includes a semi-colon (``;``) delmited header separated from a 'vaulttext' by a newline.
For example:

 .. code-block:: text

    $ANSIBLE_VAULT;1.1;V2
    eyJrZXkiOiAiZ0FBQUFBQm15bF9weGtPanc0X1NqbFBEX3ltZEk0NUhqX1hkUzBjRWc1QU1GdkJUMGZh
    eDA3UFJOeVBPS3RJUTliTHJHX3k1OWh5czBZQ25Fbi12MnRWeDFwWmdoRU94Qmx3Y1Ntd214RDRvWC1L
    eUhFMjlEVk1yUk05cmI5VGFRQjdpZGZuWWlCdVQiLCAiY2lwaGVydGV4dCI6ICJnQUFBQUFCbXlsX3A5
    MXFjVTF6dml6NFVUcC1HLU5JYVpGSG9MRU03NUEwX09GME9ZSG15MEQ1OUtST0thRDNzOFFjamtwc3Q3
    c2ZSZXhPU2ZWd0R0dU5EWGhkZjF0M3dnSnBFQzFRUGJyZE9VeHZKNFFmMnZwQT0ifQ==

or

 .. code-block:: text

    $ANSIBLE_VAULT;1.2;AES256;vault-id-label
    30613233633461343837653833666333643061636561303338373661313838333565653635353162
    3263363434623733343538653462613064333634333464660a663633623939393439316636633863
    61636237636537333938306331383339353265363239643939666639386530626330633337633833
    6664656334373166630a363736393262666465663432613932613036303963343263623137386239
    6330

The header contains up to four elements.

  1. The format ID (``$ANSIBLE_VAULT``), which currently is the only valid format ID. This ID identifies content as an Ansible Vault.

  2. The vault format version (``1.X``). All supported versions of Ansible will currently default to '1.1' or '1.2' if a labeled vault ID is supplied.
     The format version is currently used as an exact string compare only (version numbers are not currently 'compared').
     The '1.0' format was retired and is no longer supported.

  3. The encryption method used on the data (``V2`` or ``AES256``). Currently ``V2`` is the default encryption method.
      Vault format 1.0 used the ``AES`` encryption method, but it was removed and is no longer available.

  4. The vault ID label used to encrypt the data (optional, ``vault-id-label``) For example, if you encrypt a file with ``--vault-id dev@prompt``, the vault-id-label is ``dev``.

The rest of the content is the 'vaulttext'. The vaulttext is a text-armored version of the encrypted ciphertext and depends on the vault method used.
In the case of ``V2`` this is a base64 encoded string formatted to fit in 80 column lines.
For ``AES256`` this is a hexlified string formatted to fit in 80 column lines.
