# Generate SSH key Scaleway

```
# To add a new key, you can:
#   -- Add keys on your Scaleway account https://cloud.scaleway.com/#/credentials
#   -- Add keys using server tags - https://cloud.scaleway.com/#/servers/5d05550d-0b16-4a51-9414-379679c56614
#        - i.e: "AUTHORIZED_KEY=ssh-rsa_XXXXXXXXXXX AUTHORIZED_KEY=ssh-rsa_YYYYYYYYYYYYYYY"
#        - Be sure to replace all spaces with underscores
#        - $> sed 's/ /_/g' ~/.ssh/id_rsa.pub
#   -- Add the keys to '/root/.ssh/instance_keys' which will be imported
#
# And recreate your 'authorized_keys' file with the new keys:
#   -- Run 'scw-fetch-ssh-keys --upgrade'
```

# TODO
