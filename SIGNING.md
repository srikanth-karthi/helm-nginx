# Helm Chart Signing Guide

This guide explains how to sign your Helm charts for Artifact Hub.

## Prerequisites

- Helm 3.x installed
- GPG (GnuPG) installed
- A GPG key pair for signing

## Step 1: Generate a GPG Key (if you don't have one)

```bash
gpg --full-generate-key
```

Follow the prompts:
- Select RSA and RSA (default)
- Key size: 4096 bits
- Expiration: Set according to your preference
- Enter your name and email (should match the chart maintainer info)

## Step 2: List Your GPG Keys

```bash
gpg --list-secret-keys --keyid-format LONG
```

Note the key ID (the long string after `sec rsa4096/`)

## Step 3: Export Your Public Key

```bash
# Export in ASCII armor format
gpg --armor --export YOUR_EMAIL > mywebapp-pgp-public.key

# Or export using key ID
gpg --armor --export KEY_ID > mywebapp-pgp-public.key
```

## Step 4: Package and Sign Your Chart

```bash
# Package the chart
helm package mywebapp

# Sign the packaged chart
helm package mywebapp --sign --key 'YOUR_KEY_NAME' --keyring ~/.gnupg/secring.gpg
```

This will create:
- `mywebapp-0.1.0.tgz` - the packaged chart
- `mywebapp-0.1.0.tgz.prov` - the provenance file (signature)

## Step 5: Upload to Artifact Hub

1. Upload both the `.tgz` and `.tgz.prov` files to your chart repository
2. Upload the public key file to your repository
3. Update your `artifacthub-repo.yml` to reference the signing key:

```yaml
repositoryID: d6764258-4d7b-44ec-981e-48b97b2eeff0
kind: helm
name: helm-nginx
displayName: Helm Nginx Charts
url: https://raw.githubusercontent.com/srikanth-karthi/helm-nginx/main/
description: Custom Helm charts for Nginx-based web applications
owners:
  - name: srikanth-karthi
    email: srikanthkarthi2003@gmail.com
signKey:
  url: https://raw.githubusercontent.com/srikanth-karthi/helm-nginx/main/mywebapp-pgp-public.key
```

## Step 6: Verify the Signature (Optional)

```bash
helm verify mywebapp-0.1.0.tgz --keyring ~/.gnupg/pubring.gpg
```

## Automated Signing with GitHub Actions

Create `.github/workflows/release.yml`:

```yaml
name: Release Charts

on:
  push:
    branches:
      - main

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Configure Git
        run: |
          git config user.name "$GITHUB_ACTOR"
          git config user.email "$GITHUB_ACTOR@users.noreply.github.com"

      - name: Import GPG key
        run: |
          echo "${{ secrets.GPG_PRIVATE_KEY }}" | gpg --import
          echo "${{ secrets.GPG_PASSPHRASE }}" | gpg --batch --yes --passphrase-fd 0 --sign

      - name: Install Helm
        uses: azure/setup-helm@v3

      - name: Package and Sign Chart
        run: |
          helm package mywebapp --sign --key '${{ secrets.GPG_KEY_NAME }}' --keyring ~/.gnupg/secring.gpg
          helm repo index . --url https://raw.githubusercontent.com/${{ github.repository }}/main/

      - name: Commit and Push
        run: |
          git add .
          git commit -m "Release signed chart"
          git push
```

## GitHub Secrets Required

Add these secrets to your GitHub repository:
- `GPG_PRIVATE_KEY`: Your GPG private key (export with `gpg --armor --export-secret-keys YOUR_EMAIL`)
- `GPG_PASSPHRASE`: Your GPG key passphrase
- `GPG_KEY_NAME`: The name or email associated with your GPG key

## Notes

- Keep your private key secure and never commit it to the repository
- The provenance file must be uploaded alongside the chart package
- Artifact Hub will automatically detect and display the signed status
