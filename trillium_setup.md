# Connecting to Trillium (DRAC) Using VS Code

## 1. Prerequisites

- Active [Digital Research Alliance of Canada (DRAC)](https://ccdb.alliancecan.ca/) account
- [Visual Studio Code](https://code.visualstudio.com/)
- Microsoft's **Remote - SSH** VS Code extension
- SSH key registered with DRAC
- Duo two-factor authentication (MFA) enabled

## 2. Generate and register an SSH key

Open a local terminal (Git Bash on Windows):

```bash
ssh-keygen -t ed25519
```

Accept the default location (`~/.ssh/id_ed25519`).

Display your public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

Copy the public key and register it through your [DRAC account portal](https://ccdb.alliancecan.ca/).

**Important:** Upload only the `.pub` key, never the private key.

## 3. Enable two-factor authentication

1. Log in to the DRAC account portal.
2. Navigate to the multifactor authentication settings.
3. Register and activate Duo Mobile.
4. Complete the authentication setup.

## 4. Configure SSH

Open or create `~/.ssh/config` on your local computer.

On Windows, this is typically:

```text
C:\Users\<WINDOWS_USERNAME>\.ssh\config
```

Add:

```sshconfig
Host trillium
    HostName trillium.alliancecan.ca
    User <DRAC_USERNAME>
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes
```

## 5. Test the SSH connection

From your local terminal:

```bash
ssh trillium
```

Alternatively:

```bash
ssh <DRAC_USERNAME>@trillium.alliancecan.ca
```

Complete any Duo authentication prompts.

If the connection fails, use verbose mode:

```bash
ssh -v trillium
```

Look for:

```text
Offering public key
Permission denied (publickey,hostbased)
```

If the public key is rejected, verify that it has been registered with DRAC and that your account has access to Trillium.

Check your public-key fingerprint:

```bash
ssh-keygen -lf ~/.ssh/id_ed25519.pub
```

## 6. Configure VS Code

Install Microsoft's **Remote - SSH** extension.

Open the Command Palette (`Ctrl+Shift+P`) and select:

`Preferences: Open User Settings (JSON)`

Add these settings:

```json
{
    "remote.SSH.showLoginTerminal": true,
    "remote.SSH.useLocalServer": false,
    "remote.SSH.localServerDownload": "always"
}
```

These settings help display authentication prompts and avoid certain remote-server installation or startup problems.

## 7. Connect to Trillium

1. Open VS Code.
2. Press `Ctrl+Shift+P`.
3. Select `Remote-SSH: Connect to Host...`.
4. Choose `trillium`.
5. Select **Linux** when asked for the remote operating system.
6. Complete any authentication prompts.
7. Once connected, select **File → Open Folder** to access remote files.

The VS Code terminal now runs on Trillium rather than your local machine.

## 8. Troubleshooting

**SSH works, but VS Code hangs or times out:**

- Check `View → Output → Remote - SSH`.
- Confirm the settings in Step 6.
- Run `Remote-SSH: Kill VS Code Server on Host...` and reconnect.
- Look for authentication, download, server startup, or port-forwarding errors.

**Permission denied:**

- Verify your SSH username.
- Verify the registered public key.
- Check Duo MFA activation.
- Test `ssh -v trillium` from your local terminal.

## 9. Basic cluster workflow

Check the storage directories:

```bash
echo $HOME
echo $PROJECT
echo $SCRATCH
```

Typical organization:

| Directory | Purpose |
|---|---|
| `$HOME` | Code, scripts, configuration |
| `$PROJECT` | Important data and final results |
| `$SCRATCH` | Temporary computation files and intermediate results |

Use VS Code to edit code on the login node, but submit computational jobs through **Slurm**:

```bash
sbatch job.sh
```

Check job status:

```bash
squeue -u $USER
```

Cancel a job:

```bash
scancel JOB_ID
```

**Important:** Avoid running heavy computations directly on login nodes. Use allocated compute nodes through Slurm.

## References

- [Alliance Documentation](https://docs.alliancecan.ca/)
- [VS Code Remote - SSH Documentation](https://code.visualstudio.com/docs/remote/ssh)
- [DRAC Account Portal](https://ccdb.alliancecan.ca/)
