# Full Node

assumptions:

1. you're running `arm` based virtual machine
2. values that should be changed are located in `vars/definable.yaml`
3. celestia namespace is stored in plain text file and has a public url, example:

```sh
02d347f50c2ffc1ea52c
```

### Setting up the full nodes(s)

1. Create python virtual environment and install the requirements

```
python3 -m venv ansible-env
source ./ansible-venv/bin/activate
pip install ansible ansible-lint
```

2. Update the inventory file with the configurable endpoints

> You have to have direct access to the full nodes using the ssh key you will set in `ssh_key_path` variable

inventory file examples for both, [static](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html#inventory-basics-formats-hosts-and-groups) and [dynamic](https://docs.ansible.com/ansible/latest/inventory_guide/intro_dynamic_inventory.html#intro-dynamic-inventory) inventory files can be found in the `inventory_examples` directory

3. Update the `<>` values in `vars/definable.yaml`

4. Run the playbook against your inventory of nodes

```
ansible-playbook ./playbooks/fullnode.yaml -i <path-to-your-inventory>
```

5. You will get an output of the node information, similar to this:

```
TASK [Print values for manual steps and network updates] ********
ok: [3.123.143.178] => {
    "msg": [
        "Celestia namespace id: 02d347f50c2ffc1ea52c",
        "Dymint node id: 12D3KooWNyp16pAYMHkyKHkjRcSXjtt6Pd1VUGx9wC6tqesZCnNh",
        "Node public IP: 3.123.143.178",
        "Follow the manual steps to use these"
    ]
}
```

### Manual Steps

We are looking for a way to automate these steps.

1. Find the celestia block height where the returned namespace first wrote to the celestia chain.

For arabica testnet: [arabica.celenium.io](https://arabica.celenium.io)

For celestia mainnet: [celenium.io](https://celenium.io/)

2. Log into the node and update the `/etc/da/config.toml` file. Update only the pairs mentioned below, everything else should stay as is.

```
[Header]
  TrustedHash = <block-hash-of-namespace-first-write>
[DASer]
  SampleFrom = <height-of-namespace-first-write>
```

Example:

```
[Header]
  TrustedHash = "D9F81A634A98F0BAD0322E5C5A984C4CD292858B4A40C5AE631AB1FA94A0B1C1"
[DASer]
  SampleFrom = 893394
```

because namespace `02d347f50c2ffc1ea52c` first wrote to celestia arabica network at height `893394` which has the block hash of `D9F81A634A98F0BAD0322E5C5A984C4CD292858B4A40C5AE631AB1FA94A0B1C1`

3. run the services

```
sudo systemctl restart da
sudo systemctl restart rollapp
```
