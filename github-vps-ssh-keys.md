# How to set up SSH key authentication for your VPS in GitHub Actions.

### Generate a Deploy Key for Your Repository

1. Go to your GitHub repository
2. Navigate to Settings > Deploy Keys
3. Click "Add deploy key"
4. Paste the public SSH key from your VPS
5. Check "Allow write access" if needed

### Create a GitHub Secret

1. In your repository, go to Settings > Secrets and Variables > Actions
2. Click "New repository secret"
3. Create a secret called VPS_SSH_PRIVATE_KEY
4. Paste the contents of your VPS's private SSH key into this secret

## log in to your VPS to generate SSH KEYS

1. Open your terminal/command prompt on your local PC
2. SSH into your VPS:

```bash
ssh your_username@your_vps_ip
```

3. Once you're logged into the VPS, run the SSH key generation command:

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

### During the key generation:

- Press Enter to accept the default file location
- Optional: Add a passphrase for extra security
- The key will be generated in your VPS user's home directory

### After generation, you can view the public key with:

```bash
# Copy the public key to clipboard
cat ~/.ssh/id_rsa.pub
# Then go to your GitHub repository
# Settings > Deploy Keys > Add deploy key
# Paste the key and optionally allow write access
```

### As an SSH Key in your GitHub Account:

```bash
# Copy the public key to clipboard
cat ~/.ssh/id_rsa.pub
# Then go to GitHub:
# Settings > SSH and GPG keys > New SSH key
# Give it a meaningful title (e.g., "My VPS")
# Paste the key
```

### To view and copy the private key:

1. View the private key:

```bash
cat ~/.ssh/id_rsa
```

1. To copy the private key, you have a few options:

```bash
cat ~/.ssh/id_rsa | xclip -selection clipboard  # Linux
cat ~/.ssh/id_rsa | pbcopy  # macOS
```

2. For Windows (if using PuTTY or Git Bash):

```bash
cat ~/.ssh/id_rsa | clip  # Windows
```

### Important Security Warnings:

1. NEVER share your private key publicly
2. When copying to GitHub Secrets or for GitHub Actions, you'll paste the ENTIRE contents of the private key
3. The private key typically starts with -----BEGIN OPENSSH PRIVATE KEY-----
4. Ends with -----END OPENSSH PRIVATE KEY-----

### Recommended Workflow for GitHub Actions:

1. Copy the entire private key content
2. Go to your GitHub repository
3. Settings > Secrets and Variables > Actions
4. Create a new repository secret
5. Name it something like VPS_SSH_PRIVATE_KEY
6. Paste the entire private key contents

## For GitHub Actions or deployments:

1. Ensure Public Key is on VPS:

```bash
# While logged into VPS
cat ~/.ssh/authorized_keys
# Should contain your public key
```

1. Ensure the key has restricted permissions:

```bash
# On VPS
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```
