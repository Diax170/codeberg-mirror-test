# codeberg-mirror-test
Test of a GitHub -> Codeberg read-only mirror.

This approach uses an SSH deploy key which, unlike API keys, doesn't expire.

This repo already works! See the [GitHub](https://github.com/Diax170/codeberg-mirror-test) original repo and the [Codeberg](https://codeberg.org/OliMoli/codeberg-mirror-test) mirror.

# How to set up
1. Generate an SSH key pair on your machine:
```sh
mkdir /tmp/codeberg-key
ssh-keygen -t ed25519 -f /tmp/codeberg-key/key -C github-actions
```
2. In your GitHub repository, navigate to: _Settings -> Secrets and variables -> Actions -> New repository secret_.
3. Name the secret _CODEBERG_SSH_KEY_ and paste the complete contents of the **private** SSH key you've just generated (at `/tmp/codeberg-key/key`) into the _Secret_ field.
4. Click _Add secret_.
5. In your Codeberg repository, navigate to: _Settings -> Deploy keys -> Add deploy key_.
6. Name the key _GitHub Actions_ and paste the contents of the **public** SSH key (at `/tmp/codeberg-key/key.pub`) into the _Content_ field.
7. Make sure to **check the _Enable write access_ box!**
8. Click _Add deploy key_.
9. Clone the GitHub repository and create the file `.github/workflows/codeberg-mirror.yaml`:
```yaml
name: Codeberg mirror

on:
  push:
    branches:
      - "**"

jobs:
  push:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v7
        with:
          fetch-depth: 0

      - name: Add SSH key
        env:
          SSH_AUTH_SOCK: /tmp/ssh_agent.sock
        run: |
          mkdir -p ~/.ssh
          ssh-keyscan codeberg.org >> ~/.ssh/known_hosts
          echo "${{ secrets.CODEBERG_SSH_KEY }}" > ~/.ssh/github_actions
          chmod 600 ~/.ssh/github_actions
          ssh-agent -a $SSH_AUTH_SOCK > /dev/null	
          ssh-add ~/.ssh/github_actions

      - name: Push to Codeberg
        env:
          SSH_AUTH_SOCK: /tmp/ssh_agent.sock
        run: |
          # NOTE: make sure to adjust your own Codeberg URL!
          git remote add codeberg ssh://git@codeberg.org/OliMoli/codeberg-mirror-test
          git push codeberg --force
```
10. Once you push the changes, they should automatically appear in the Codeberg mirror.
11. You can now remove the local SSH key pair:
```sh
rm -r /tmp/codeberg-key
```
