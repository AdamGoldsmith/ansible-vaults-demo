# ansible-vaults-demo

I made this repo to demonstrate how to create a multi-staged Ansible environment with multiple ansible-vaulted variables referenced by ansible vault ids.

## General configuration

1. Two inventory environment directories were configured under `inv.d` called production and test (by default test is referenced in `ansible.cfg`).
1. Two encryption/decyption password files called `.prod.pass` & `.test.pass` were created in the `vault-ids` directory (see the [warning](#warning) below).
1. The environments encryption/decryption password files are referenced in `ansible.cfg` using the `vault_identity_list` key. The decryption process will work through this list until it finds a key that works or the list is exhausted.
    ```ini
    vault_identity_list = example-test@vault-ids/.test.pass,example-prod@vault-ids/.prod.pass
    ```
1. There is an encrypted variable called `db_password` (stored in each inventory's `group_vars` directory) that has been encrypted using the following commands for each environment:

    ```bash
    ansible-vault encrypt_string --encrypt-vault-id example-test '123450' --name 'db_password'db_password: !vault |
            $ANSIBLE_VAULT;1.2;AES256;example-test
            30353231613763613833626630363636343031323331326431393033653239336161646265393030
            3434613130376265393738656135636561656164306565640a316661393931336164353862363730
            36376632663239303464336437386364393130303838343636666339313731626166333737353365
            3538323038623464630a356230663533363931333435633936393832396264363433313361636132
            3461
    Encryption successful

    ansible-vault encrypt_string --encrypt-vault-id example-prod '567890' --name 'db_password'
    db_password: !vault |
            $ANSIBLE_VAULT;1.2;AES256;example-prod
            61633730393562646635306162336138323464353831323230343835333030383833636239613065
            6239303366656332633631353361373861613539343735640a336532313334646161643631333237
            66383537653065343462633331666232636362373461323132663731396236373335336336363530
            3665643061313834330a363633636163396364336339366433316261653030316334616137666166
            6535
    Encryption successful
    ```
     The encrypted value output to the screen was pasted into each environment's associated inventory group vars here:

    Environment | Inventory file |
    -----------:|--------------- |
    test        | `inv.d/test/group_vars/secure_vars.yml` |
    production  | `inv.d/production/group_vars/secure_vars.yml` |

## Using the variables

To test decrypting the variables for each environment, perform the following:

```bash
ANSIBLE_INVENTORY=inv.d/test ansible localhost -m debug -a "var=db_password"
localhost | SUCCESS => {
    "db_password": "123456"
}

ANSIBLE_INVENTORY=inv.d/production ansible localhost -m debug -a "var=db_password"
localhost | SUCCESS => {
    "db_password": "567890"
}
```

## WARNING

The ansible-vault encrypt/decrypt passwords are stored inside this repo as it's purely for demonstration purposes - this is utterly stupid in a real-life scenario so remember to create these files outside the repo and update the reference to them in `ansible.cfg` here:

```ini
vault_identity_list = example-test@vault-ids/.test.pass,example-prod@vault-ids/.prod.pass
```

