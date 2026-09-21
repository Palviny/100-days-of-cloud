## Day 01 — Create an Azure SSH Key Pair

## Objective

Create an SSH key pair in Microsoft Azure according to the following requirements:

- Key pair name: `xfusion-kp`
- Key type: `RSA`

## Environment

- Cloud Provider: Microsoft Azure
- Service: Azure SSH Keys
- Key Pair Name: `xfusion-kp`
- Key Type: RSA

## Implementation

### Step 1 — Open Azure SSH Keys

I opened the Azure Portal and searched for **SSH keys**.
<img width="1265" height="380" alt="image" src="https://github.com/user-attachments/assets/b79b34cb-95f3-41be-82bc-98ea81441a62" />

### Step 2 — Create the SSH Key

I selected **Create** and configured the SSH key with the required values:
<img width="907" height="701" alt="image" src="https://github.com/user-attachments/assets/4e094d48-9868-4248-8a86-e8a948f13c0a" />



### Step 3 — Create

I reviewed the configuration and created the SSH key pair.
<img width="1908" height="116" alt="image" src="https://github.com/user-attachments/assets/09c05e67-3477-4924-83b7-ef747d33581a" />

## Security Notes

The private SSH key was not uploaded to GitHub or included in this repository.

Private SSH keys should be treated as sensitive credentials and stored securely.

## Key Takeaways

- Azure provides managed SSH key resources that can be used when deploying Linux virtual machines.
- RSA is one of the supported SSH key types.
- Private keys must be protected and should never be committed to a public Git repository.
