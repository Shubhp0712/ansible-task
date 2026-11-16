# Ansible NGINX Deployment

This project uses an Ansible role to install and start NGINX on an AWS EC2
instance, create the configured users, and deploy a custom static HTML page.

## Project structure

```text
.
├── ansible.cfg
├── inventory.ini
├── main.yaml
├── nginx-role/
│   ├── defaults/main.yml
│   ├── files/index.html
│   ├── handlers/main.yml
│   ├── meta/main.yml
│   ├── tasks/main.yml
│   ├── tests/
│   │   ├── inventory
│   │   └── test.yml
│   ├── vars/main.yml
│   └── README.md
└── .gitignore
```

## Prerequisites

- Ansible installed on the control machine.
- An AWS EC2 instance running a supported RPM-based Linux distribution, such
  as Amazon Linux.
- The EC2 security group allows inbound SSH (`22`) and HTTP (`80`). Allow
  HTTPS (`443`) if HTTPS will also be configured.
- The EC2 private key (`.pem`) is available locally.
- The EC2 instance has a public IP address and is reachable from the control
  machine.

## Configure the SSH key

Place the private key in the project directory and restrict its permissions:

```bash
chmod 400 <key-file>.pem
```

Private keys must not be committed to source control. The repository's
`.gitignore` excludes `.pem` files.

## Configure `inventory.ini`

Update `inventory.ini` with the EC2 public IP, the correct remote username,
and the local private-key filename:

```ini
ec2_instance ansible_host=<EC2_PUBLIC_IP> ansible_user=<EC2_USERNAME> ansible_ssh_private_key_file=<key-file>.pem
```

For Amazon Linux, the username is commonly `ec2-user`. The username depends on
the AMI, so use the account configured by the selected AMI.

## Configure `ansible.cfg`

Create `ansible.cfg` in the project root (the same directory as `main.yaml`)
with the following contents:

```ini
[defaults]
inventory = <Path of inventory file>
roles_path = <Path of your Role directory>
host_key_checking = False 
```

This configuration makes Ansible use `inventory.ini` automatically, searches
the project root for the `nginx-role` directory, and prevents an interactive
host-key prompt on the first connection. For production environments, consider
leaving host-key checking enabled.

Because `ansible.cfg` is in the project root, Ansible automatically discovers
it when commands are run from this directory. You can also select it
explicitly:

```bash
ANSIBLE_CONFIG=./ansible.cfg ansible-playbook main.yaml
```

## Run the deployment

Run these command from the project :

```bash
ansible-playbook main.yaml
```

The playbook installs NGINX, starts the service, creates the users listed in
`nginx-role/vars/main.yml`, and copies `nginx-role/files/index.html` to the
NGINX document root.

After a successful run, open the EC2 public IP in a browser:

```text
http://<EC2_PUBLIC_IP>
```

## Customize the deployment

- Change the users in `nginx-role/vars/main.yml`.
- Replace `nginx-role/files/index.html` with the page to be served.
- Change `src`, `dest`, or `package_name` in `nginx-role/vars/main.yml` when
  deploying to a different package or document root.

## Troubleshooting

- **Permission denied for the key:** run `chmod 400 <key-file>.pem`.
- **Connection timeout:** verify the EC2 public IP, route, instance state, and
  security-group SSH rule.
- **Authentication failure:** verify `ansible_user` for the AMI and the
  private-key filename in `inventory.ini`.
- **HTTP is unreachable:** verify the security-group HTTP rule and that the
  NGINX service is running.