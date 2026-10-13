# GitHub Access from Trillium (DRAC)

This guide configures GitHub SSH authentication **on Trillium** so you can clone private repositories and push changes from the VS Code remote terminal.

> Your DRAC/Trillium SSH credentials are separate from your GitHub credentials. Run the commands below **on Trillium**, not on your local computer.

## 1. Generate a dedicated GitHub SSH key

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_github -C "trillium-github"
```

Set a passphrase when prompted. This creates:

- `~/.ssh/id_ed25519_github` — **private key; never share it**
- `~/.ssh/id_ed25519_github.pub` — public key, safe to register with GitHub

## 2. Add the public key to GitHub

Display the key:

```bash
cat ~/.ssh/id_ed25519_github.pub
```

Copy the full output. Open [GitHub → Settings → SSH and GPG keys](https://github.com/settings/keys), select **New SSH key**, choose **Authentication Key**, name it `Trillium`, and paste the public key.

## 3. Configure SSH on Trillium

Edit `~/.ssh/config` (create it if needed):

```bash
mkdir -p ~/.ssh
nano ~/.ssh/config
```

Add this block, preserving any other hosts already in the file:

```sshconfig
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_github
    IdentitiesOnly yes
```

Set appropriate permissions:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/config ~/.ssh/id_ed25519_github
```

## 4. Test GitHub authentication

```bash
ssh -T git@github.com
```

On the first connection, **verify GitHub's SSH host-key fingerprint** against [GitHub's published fingerprints](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints) before accepting it.

Expected success message (the lack of shell access is normal):

```text
Hi YOUR_GITHUB_USERNAME! You've successfully authenticated, but GitHub does not provide shell access.
```

## 5. Clone a repository

Keep source code in your home directory:

```bash
mkdir -p ~/projects
cd ~/projects
git clone git@github.com:YOUR_USERNAME/YOUR_REPO.git
cd YOUR_REPO
```

In the VS Code window connected to Trillium, use **File → Open Folder** and open `~/projects/YOUR_REPO`.

For a *public* repository that you only need to download, SSH setup is unnecessary:

```bash
git clone https://github.com/YOUR_USERNAME/YOUR_REPO.git
```

## 6. Everyday Git commands

```bash
git status
git pull
git add .
git commit -m "Describe your change"
git push
```

For your first commit on the cluster, you may need to configure your Git identity:

```bash
git config --global user.name "Your Name"
git config --global user.email "your-github-email@example.com"
```

## Troubleshooting

- **`Permission denied (publickey)`**: Check that you registered the correct `.pub` key and that `~/.ssh/config` names the corresponding private key. Run `ssh -vT git@github.com` for details.
- **`Repository not found`**: Verify the repository owner/name and that your GitHub account has access. Organization repositories may require additional SSO authorization.
- **Passphrase prompts**: These are for your GitHub *private key*, not your DRAC password or Duo MFA.

**Storage reminder:** Keep code in `$HOME`; use `$SCRATCH` for large temporary outputs and `$PROJECT` for important results, subject to your allocation and cluster policies.

## References

- [GitHub: Connecting to GitHub with SSH](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
- [GitHub: SSH key fingerprints](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints)
- [Alliance documentation](https://docs.alliancecan.ca/)
