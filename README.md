# Git-Multi-User-with-SSH-and-Remotes

This guide walks you through setting up multiple Git identities with SSH signing and remotes. It covers generating SSH keys, configuring per-identity Git settings (including SSH commit signing), and connecting those identities to hosting services such as GitHub.

## Prerequisites

### Generating an SSH key pair

Run the following (recommended) command to create an ed25519 key for each identity. Use `-f` to set a custom filename so keys don't overwrite each other:

```bash
ssh-keygen -t ed25519 -C "email@example.com" -f ~/.ssh/id_ed25519_identity
```

Follow the prompts and choose a passphrase you can remember (recommended). If you prefer no passphrase, press Enter when prompted.

Notes:
- Keep the private key (e.g. `~/.ssh/id_ed25519_identity`) secret. The public key is the one you upload to services.
- When using SSH commit signing, you will register the *public* key as a signing key on the hosting service.

## Configuring Git identities and SSH commit signing

You can store multiple identities in your global `~/.gitconfig` under different `user` subsections and switch between them per-repository.

Example: add one identity block for each identity in your global config (or use the `git config` commands shown below):

```ini
[user "work"]
	name = Work Name
	email = work@example.com
	signingkey = ~/.ssh/id_ed25519_work.pub

[user "personal"]
	name = Personal Name
	email = personal@example.com
	signingkey = ~/.ssh/id_ed25519_personal.pub
```

You can set those values from the command line as well. Note: these `user.<key>.*` names are sections in the Git config and are not the active commit identity until you copy them into `user.name`/`user.email` for a repo.

```bash
# Example (sets values in global config under a named subsection):
git config --global user.work.name "Work Name"
git config --global user.work.email "work@example.com"
git config --global user.work.signingkey "~/.ssh/id_ed25519_work.pub"
```

To tell Git to use SSH as the commit-signing backend:

```bash
git config --global gpg.format ssh
```

Important: `user.signingkey` should point to your public key file (for example `~/.ssh/id_ed25519_work.pub`) — the public key is what you upload to the hosting service as a Signing Key. The private key is used locally (via your SSH agent) to create the signature.

If you want Git to refuse to make commits unless `user.name` and `user.email` are explicitly set in the repository (safer when managing multiple identities), enable:

```bash
git config --global user.useConfigOnly true
```

To enable automatic commit signing by default:

```bash
git config --global commit.gpgsign true
```

### Aliases to switch identities

On Unix-like shells (Git Bash, WSL, macOS Terminal) you can create a Git alias that copies values from a named subsection into the active `user.*` for the current repo.

```bash
git config --global alias.identity '!sh -c "git config user.name \"$(git config user.$1.name)\" && git config user.email \"$(git config user.$1.email)\" && git config user.signingkey \"$(git config user.$1.signingkey)\"" -'
```

Usage (in a repo):

```bash
git identity work
```

PowerShell alternative (Windows users):

```powershell
function Set-GitIdentity {
  param([string]$Identity)
  git config user.name (git config --get "user.$Identity.name")
  git config user.email (git config --get "user.$Identity.email")
  git config user.signingkey (git config --get "user.$Identity.signingkey")
}
```
Usage (in a repo):

```powershell
Set-GitIdentity work
```

Add the PowerShell function to your PowerShell profile (`$PROFILE`) if you want it available in every session.

## Connecting identities to hosting services

### SSH host entries

In `~/.ssh/config` add an entry per identity/host to select which private key to use when connecting. Use the private key file (not the .pub) in `IdentityFile`:

```sshconfig
Host github-work
	HostName github.com
	PreferredAuthentications publickey
	IdentityFile ~/.ssh/id_ed25519_work

Host github-personal
	HostName github.com
	PreferredAuthentications publickey
	IdentityFile ~/.ssh/id_ed25519_personal

Host gitlab-personal
	HostName gitlab.com
	PreferredAuthentications publickey
	IdentityFile ~/.ssh/id_ed25519_personal
```

You can then use `git@github-work:owner/repo.git` as the remote to force that key.

### GitHub: Add SSH keys

1. Go to **Settings → Access → SSH and GPG keys** and add a new SSH key with the following properties:

   - **Title**: Name the connection
   - **Key Type**: Authentication Key
   - **Key**: Paste the public SSH key

2. Add another SSH key:

   - **Title**: Name the connection
   - **Key Type**: Signing Key
   - **Key**: Paste the public SSH key

### GitHub: Add SSH keys

1. Go to **Edit profile → SSH Keys** and add a new SSH key with the following properties:

   - **Title**: Name the connection
   - **Usage type**: Authentication & Signing
   - **Key**: Paste the public SSH key

### Verify connection

Verify that your SSH key was added correctly:

```bash
ssh -T git@gitlab-personal
```

## Tips

To avoid re-entering a passphrase every time, add your private key to an agent for the session:

```bash
ssh-add ~/.ssh/id_ed25519_work
```

## Usage

In a repository, to switch to an identity (Unix shell example):

```bash
git identity work
```

Or in PowerShell after adding the helper function:

```powershell
Set-GitIdentity work
```

This copies the named subsection (`user.work.*`) into the local repo's `user.name` / `user.email` / `user.signingkey` values.

## References

- Micah Henning - [Setting Up Git Identities](https://www.micah.soy/posts/setting-up-git-identities/)
- GitHub Docs - [Generating a new SSH key and adding it to the ssh-agent](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
- GitHub Docs - [Adding a new SSH key to your GitHub account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)
- GitLab Docs - [Use SSH keys to communicate with GitLab](https://docs.gitlab.com/user/ssh/)
