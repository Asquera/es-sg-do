## Install Dependencies

You'll need `openssl`, `python`, `pip`, `java` and `virtualenv` installed. If you have
them, skip the rest of this section. If not, read on.

You probably already have `python` which generally includes either `pip` or
`easy_install`. I can almost guarantee you have `openssl` or something similar
installed. Install `java` however your distro asks.

If your system already has `pip` you don't need to install it, obviously! If
not, install it with the following:

```bash
sudo easy_install pip
```

Then you can install `virtualenv` with `pip`:

```bash
sudo pip install virtualenv
```

Next set up your Digital Ocean API key:

```bash
cp digital_ocean.ini.SAMPLE digital_ocean.ini
```

Replace `CHANGE_THIS` with the a correct API key.

## Prepare a Virtual Environment

In order to properly provision and setup the machine we need to install some
`python` packages like `ansible` and `dopy`. Thankfully `pip` and `virtualenv`
make this quite convenient.

First we source the environment to 'enter' it, then we use `pip` to install the
required packages:

```bash
virtualenv env
source env/bin/activate
pip install -r requirements.txt
```

# Provisioning

First enter the virtual environment like before if you aren't in it anymore:

```bash
source env/bin/activate
```

Then to provision or update the machine you can run `ansible-playbook` like so:

```bash
env/bin/ansible-playbook provision.yml
```

# Vacating

In order to rebuild you can vacate the cluster, destroying the nodes and PKI:

```bash
env/bin/ansible-playbook vacate.yml
```

# Testing

> By default the scripts configure the cluster for testing. You can set `- testing: false` in the `settings.yml` to disable this.

All certs and keys are injected into machines in the `/root/certs/` directory. Log in with the `root` user and you can make requests like so:

```bash
# As Admin
curl -k --cacert certs/chain-ca.pem --cert certs/admin.crt.pem --key certs/admin.key.pem "https://0.0.0.0:9200/_searchguard/authinfo" | jq
# As user
curl -k --cacert certs/chain-ca.pem --cert certs/user.crt.pem --key certs/user.key.pem "https://0.0.0.0:9200/_searchguard/authinfo" | jq
```

The `admin` user is able to do anything.

```bash
# This will succeed:
curl -k --cacert certs/chain-ca.pem --cert certs/admin.crt.pem --key certs/admin.key.pem "https://0.0.0.0:9200/movies/_search?q=*:*" | jq
# This will succeed:
curl -k --cacert certs/chain-ca.pem --cert certs/admin.crt.pem --key certs/admin.key.pem -X POST --data-binary @item_seed.json 'https://localhost:9200/movies/_bulk' | jq
# This will succeed:
curl -k --cacert certs/chain-ca.pem --cert certs/admin.crt.pem --key certs/admin.key.pem "https://0.0.0.0:9200/users/_search?q=*:*" | jq
```

The `user` user is only allowed to read from the movies index.

```bash
# Can only read from Movies. This will succeed:
curl -k --cacert certs/chain-ca.pem --cert certs/user.crt.pem --key certs/user.key.pem "https://0.0.0.0:9200/movies/_search?q=*:*" | jq
# Can't write to movies. This will fail:
curl -k --cacert certs/chain-ca.pem --cert certs/user.crt.pem --key certs/user.key.pem -X POST --data-binary @item_seed.json 'https://localhost:9200/movies/_bulk'
# Can't read from users. This will fail:
curl -k --cacert certs/chain-ca.pem --cert certs/user.crt.pem --key certs/user.key.pem "https://0.0.0.0:9200/users/_search?q=*:*" | jq
```

