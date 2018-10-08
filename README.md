vault-salt
===

This script simply rotates the vault token for a salt master using the [vault-shim](https://github.com/ev0rtex/vault-shim) script. To do this automatically on an interval just set up a cron job:

```
0 0 1 * * /opt/vault-salt/rotate_token 2>&1 >>/var/log/salt_token.log && systemctl restart salt-master
```

**NOTE:** This script assumes you have the vault-shim repo cloned at `../vault-shim` currently.

### Vault requirements

In order to use this correctly you will need to use approle's in vault:

```sh
$ vault auth enable approle
$ vault write auth/approle/role/salt-master \
secret_id_ttl=1m \
secret_id_num_uses=1 \
token_num_uses=0 \
token_ttl=768h \
token_max_ttl=0 \
policies=salt-master
$ vault read auth/approle/role/salt-master/role-id
Key        Value
---        -----
role_id    14bc0c59-fdcb-5798-5406-d48aec3bf7ba
$ vault write -f auth/approle/role/salt-master/secret-id
Key                   Value
---                   -----
secret_id             4ea68443-84b2-605c-e3b6-a6e6cfe81a03
secret_id_accessor    5f7e6f1e-7d0b-a38e-d31a-76def56f724e
$ vault write auth/approle/login \
role_id=14bc0c59-fdcb-5798-5406-d48aec3bf7ba \
secret_id=4ea68443-84b2-605c-e3b6-a6e6cfe81a03
Key                     Value
---                     -----
token                   3ea99c29-f284-2577-7c14-698a56cf573c
token_accessor          82dc5a6b-925c-578c-0419-a6cdec41e74b
token_duration          768h
token_renewable         true
token_policies          ["default" "salt-master"]
identity_policies       []
policies                ["default" "salt-master"]
token_meta_role_name    salt-master
```

