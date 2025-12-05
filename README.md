# Git-Multi-User-with-SSH-and-Remotes

This guide walks you through setting up multiple Git identities with SSH signing and remotes. It covers generating SSH keys, configuring per-identity Git settings (including SSH commit signing), and connecting those identities to hosting services such as GitHub.

## Setup

If you want Git to refuse to make commits unless you have explicitly set your identity in the repository (safer when managing multiple identities), enable:

```bash
git config --global user.useConfigOnly true
```

### SSH Signing

To tell Git to use SSH as the commit-signing backend:

```bash
git config --global gpg.format ssh
```

To enable automatic commit signing by default:

```bash
git config --global commit.gpgsign true
```

### Aliases to switch identities

On Unix-like shells (Git Bash, WSL, macOS Terminal) you may create a Git alias that sets the identity for the current repository:

```bash
git config --global alias.identity '!sh -c "git config user.name \"$(git config user.$1.name)\" && git config user.email \"$(git config user.$1.email)\" && git config user.signingkey \"$(git config user.$1.signingkey)\"" -'
```

Usage (in a repo):

```bash
git identity <identifier>
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
Set-GitIdentity <identifier>
```

Add the PowerShell function to your PowerShell profile (`$PROFILE`) if you want it available in every session.

## Creating identities

An identity has

- an identifier (e.g. `work`, `personal`)
- a name (e.g. `Mike`)
- an email (e.g. `mike@business.com`, `mike@personal.dev`)
- a private (e.g. `~/.ssh/id_ed25519`) and a public (e.g. `~/.ssh/id_ed25519.pub`) key

### Generating an SSH key pair

Run the following command to create an ed25519 (recommended) key for each identity. Use `-f` to set a custom filename so keys don't overwrite each other:

```bash
ssh-keygen -t ed25519 -C <email> -f <path/to/key>
```

Follow the prompts and choose a passphrase you can remember (recommended). If you prefer no passphrase, press Enter when prompted.

> [!important]
> Keep the private key (`path/to/key`) secret. The public key (`path/to/key.pub`) may be shared with anyone.

### Configuring an identity

Create an identity with name, email and signing key:

```bash
git config --global user.<identifier>.name <name>
git config --global user.<identifier>.email <email>
git config --global user.<identifier>.signingkey <path/to/key.pub>
```

## Connecting to hosting services

To connect with hosting services, an identity gets a `connection` per remote.

### SSH host entries

In `~/.ssh/config` add an entry per connection (identity/host) to select which private key to use when connecting. Use the private key file in `IdentityFile`:

```sshconfig
Host <connection>
	HostName <host ip or domain>
	PreferredAuthentications publickey
	IdentityFile <path/to/key>
```

To now reference a repository (e.g. for cloning), use `git@<connection>:owner/repo.git` as the remote.

### Popular hosting services

#### GitHub: Add SSH keys

1. Go to **Settings → Access → SSH and GPG keys** and add a new SSH key with the following properties:

   - **Title**: Name the connection
   - **Key Type**: Authentication Key
   - **Key**: Paste the public SSH key

2. Add another SSH key:

   - **Title**: Name the connection
   - **Key Type**: Signing Key
   - **Key**: Paste the public SSH key

#### GitLab: Add SSH keys

1. Go to **Edit profile → SSH Keys** and add a new SSH key with the following properties:

   - **Title**: Name the connection
   - **Usage type**: Authentication & Signing
   - **Key**: Paste the public SSH key

#### Codeberg: Add SSH keys

1. Go to **Settings → SSH / GPG Keys** and add a new SSH key with the following properties:

   - **Key name**: Name the connection
   - **Content**: Paste the public SSH key

### Verify connection

Verify that your SSH key was added correctly:

```bash
ssh -T git@<connection>
```

## Tips

To avoid re-entering a passphrase every time, add your private key to an agent for the session:

```bash
ssh-add <path/to/key>
```

## Example

In this example, two identities will be created:

- Mike's business identity:
  - identifier: `business`
  - name: `Mike`
  - email: `mike@business.com`
  - private key at `~/.ssh/id_25519_business`
  - public key at `~/.ssh/id_25519_business.pub`
  - connection `github-business` to GitHub
- Mike's personal identity:
  - identifier: `personal`
  - name: `Mike`
  - email: `mike@personal.dev`
  - private key at `~/.ssh/id_25519_personal`
  - public key at `~/.ssh/id_25519_personal.pub`
  - connection `github-personal` to GitHub
  - connection `gitlab-personal` to GitLab

### Generating SSH key pairs

Run the following:

```bash
ssh-keygen -t ed25519 -C mike@business.com -f ~/.ssh/id_25519_business
ssh-keygen -t ed25519 -C mike@personal.dev -f ~/.ssh/id_25519_personal
```

### Configuring identities

Run the following:

```bash
git config --global user.business.name "Mike"
git config --global user.business.email "mike@business.com"
git config --global user.business.signingkey "~/.ssh/id_25519_business.pub"
git config --global user.personal.name "Mike"
git config --global user.personal.email "mike@personal.dev"
git config --global user.personal.signingkey "~/.ssh/id_25519_personal.pub"
```

### Connecting to hosting services

Add the following to your SSH config file:

```sshconfig
Host github-business
	HostName github.com
	PreferredAuthentications publickey
	IdentityFile ~/.ssh/id_25519_business
Host github-personal
	HostName github.com
	PreferredAuthentications publickey
	IdentityFile ~/.ssh/id_25519_personal
Host gitlab-personal
	HostName gitlab.com
	PreferredAuthentications publickey
	IdentityFile ~/.ssh/id_25519_personal
```

In the business **GitHub** account:

1. **Settings → Access → SSH and GPG keys** and add a new SSH key:

   - **Title**: Mike's PC
   - **Key Type**: Authentication Key
   - **Key**: Content of ~/.ssh/id_25519_business.pub

2. Add another SSH key:

   - **Title**: Mike's PC
   - **Key Type**: Signing Key
   - **Key**: Content of ~/.ssh/id_25519_business.pub

In the personal **GitHub** account:

1. **Settings → Access → SSH and GPG keys** and add a new SSH key:

   - **Title**: Mike's PC
   - **Key Type**: Authentication Key
   - **Key**: Content of ~/.ssh/id_25519_personal.pub

2. Add another SSH key:

   - **Title**: Mike's PC
   - **Key Type**: Signing Key
   - **Key**: Content of ~/.ssh/id_25519_personal.pub

In the personal **GitLab** account:

1. **Edit profile → SSH Keys** and add a new SSH key:

   - **Title**: Mike's PC
   - **Usage type**: Authentication & Signing
   - **Key**: Content of ~/.ssh/id_25519_personal.pub

## References

- Micah Henning - [Setting Up Git Identities](https://www.micah.soy/posts/setting-up-git-identities/)
- GitHub Docs - [Generating a new SSH key and adding it to the ssh-agent](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
- GitHub Docs - [Adding a new SSH key to your GitHub account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)
- GitLab Docs - [Use SSH keys to communicate with GitLab](https://docs.gitlab.com/user/ssh/)
- Codeberg Docs - [Adding an SSH key to your account](https://docs.codeberg.org/security/ssh-key/)
