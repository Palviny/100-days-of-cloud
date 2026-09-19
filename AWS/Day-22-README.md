# Day 22 — Configuring Secure SSH Access to an EC2 Instance

## Objective

The Nautilus DevOps team needed to configure a new EC2 instance
that could be accessed securely from the `aws-client` landing host.

The requirements were:

- Create an EC2 instance named `nautilus-ec2`
- Use instance type `t2.micro`
- Create an SSH key named `id_rsa`
- Store the key under `/root/.ssh/` on `aws-client`
- Use the key to access the EC2 instance
- Configure the key for the `root` user
- Verify passwordless SSH access from `aws-client`

> Lab credentials were provided by the lab environment and were
> not stored in this repository.

---

## The Problem

An EC2 instance needs a secure way for administrators to connect
remotely.

Instead of using a password, SSH can authenticate users using
public-key cryptography.

For this lab, the private SSH key remains on the `aws-client`
host while the corresponding public key is installed on the
EC2 instance.

---

## The Solution

We used an SSH key pair:

- `id_rsa` : private key
- `id_rsa.pub` : public key

The private key remains on the `aws-client` host and must be kept
secret.

The public key is installed on the EC2 instance in the user's
SSH `authorized_keys` file.

The EC2 instance was launched with the `id_rsa` key pair, allowing
the initial `ec2-user` account to authenticate using SSH.

The public key was then added to the `root` user's
`authorized_keys` file so that root could also authenticate
without a password.

---

## How SSH Public-Key Authentication Works

The basic flow is:

```text
aws-client                              EC2 instance
──────────                              ────────────

id_rsa 
(private key)

id_rsa.pub   ──────────────────────►  authorized_keys
                                      (public key)

SSH authentication proves possession of the private key.

Implementation
Step 1: Check whether an SSH key already exists

On the aws-client host: ls -l /root/.ssh/id_rsa*

Command breakdown
ls → lists files
-l → displays detailed file information
/root/.ssh/ → root user's SSH directory
id_rsa* → matches files beginning with id_rsa

We were looking for:
/root/.ssh/id_rsa
/root/.ssh/id_rsa.pub

If the key did not exist, it was created.

Step 2: Generate the SSH key pair
ssh-keygen -t rsa -b 4096 -f /root/.ssh/id_rsa

Command breakdown
ssh-keygen → creates an SSH key pair
-t rsa → use the RSA key algorithm
-b 4096 → create a 4096-bit key
-f /root/.ssh/id_rsa → specify where the key should be stored

This creates two files:

id_rsa       → private key
id_rsa.pub   → public key

The private key must remain protected.

Step 3: Import the public key into AWS
aws ec2 import-key-pair \
  --key-name id_rsa \
  --public-key-material fileb:///root/.ssh/id_rsa.pub \
  --region us-east-1

Command breakdown
aws → AWS Command Line Interface
ec2 → interact with Amazon EC2
import-key-pair → import an existing public SSH key
--key-name id_rsa → name the EC2 key pair
--public-key-material → provide the public key
fileb:///root/.ssh/id_rsa.pub → read the public key from the local file
--region us-east-1 → specify the AWS region

Only the public key is imported into AWS.

The private key remains on aws-client.

Step 4: Launch the EC2 instance

The EC2 instance was configured with:

Setting	Value
Name:	nautilus-ec2
Instance type:	t2.micro
Operating system:	Amazon Linux
Key pair: id_rsa
SSH	TCP: port 22

The id_rsa key pair was selected during instance launch.

This is important because the EC2 key pair is used to place
the public key into the initial user's SSH configuration.

Step 5: Connect using the initial EC2 user

After the instance reached the Running state and passed its
status checks, its public IP address was obtained.

SSH was used from aws-client:

ssh -i /root/.ssh/id_rsa ec2-user@<EC2-IP>

Command breakdown
ssh → starts an SSH connection
-i /root/.ssh/id_rsa → tells SSH which private key to use
ec2-user → initial Amazon Linux user
<EC2-IP> → public IPv4 address of the EC2 instance

At this point, the private key remains on aws-client while
SSH uses it to authenticate against the public key installed
on the EC2 instance.

Step 6: Configure SSH access for root

Once connected as ec2-user, the root SSH directory was created:
sudo mkdir -p /root/.ssh

Command breakdown
sudo → execute with administrator privileges
mkdir → create a directory
-p → create missing parent directories and avoid errors
if the directory already exists
/root/.ssh → root user's SSH configuration directory

Step 7: Copy the authorized key
sudo cp /home/ec2-user/.ssh/authorized_keys /root/.ssh/authorized_keys

Command breakdown
sudo → execute with administrator privileges
cp → copy a file
/home/ec2-user/.ssh/authorized_keys → current user's
authorized public keys
/root/.ssh/authorized_keys → root user's authorized keys

This allows root to trust the same public key.

Step 8: Secure the SSH directory
sudo chmod 700 /root/.ssh

chmod changes file and directory permissions.

700 means:
Owner:  read + write + execute
Group:  no permissions
Others: no permissions

Step 9: Secure the authorized keys file
sudo chmod 600 /root/.ssh/authorized_keys

600 means:
Owner:  read + write
Group:  no permissions
Others: no permissions

Restrictive permissions help protect SSH authentication files.

Step 10: Set the correct ownership
sudo chown -R root:root /root/.ssh

Command breakdown
chown → change ownership
-R → apply recursively
root:root → root user and root group
/root/.ssh → directory whose ownership is changed

The SSH configuration now belongs to root.

Step 11: Exit the EC2 instance
exit

This closes the SSH session and returns to the aws-client
terminal.

Step 12 — Verify passwordless root SSH access

From aws-client:

ssh -i /root/.ssh/id_rsa root@<EC2-IP>

If successful, the connection is established without entering
an account password.

Verification

The final test confirmed that:

The private key remained on aws-client
The public key was trusted by the EC2 instance
The root user's authorized_keys contained the key
SSH permissions were correctly configured
Root could authenticate using the SSH key

Real-World Relevance

SSH is widely used for remote administration of Linux servers.

In cloud environments, administrators may use SSH to:

Troubleshoot servers
Install and configure software
Inspect logs
Manage applications
Perform system administration
Diagnose networking problems

However, production environments often restrict direct SSH access
and may use services such as AWS Systems Manager Session Manager
to reduce exposure of SSH to the internet.

Key Concepts Learned
SSH
Public/private key cryptography
EC2 key pairs
authorized_keys
Linux file permissions
chmod
chown
sudo
AWS CLI
EC2 authentication
Passwordless authentication

What I Learned

The most important lesson from this lab was understanding the
difference between creating an SSH key and actually
configuring a server to trust that key.

Creating id_rsa on the client alone is not enough.

The public key must be trusted by the EC2 instance, while the
private key stays securely on the client.

I also learned that SSH access depends on more than just the key:
network access through port 22, the correct Linux username,
the key pair configuration, and correct file permissions all
matter.
