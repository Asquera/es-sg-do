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

