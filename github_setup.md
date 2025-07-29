# 🔧 Git & GitHub Setup Guide (Linux)

This guide will help you set up Git and connect your local Linux machine to GitHub using SSH. It's perfect for fresh environments or reconfiguring your workstation.

---

## 📦 1. Install Git

Update your package list and install Git:

```bash
sudo apt update
sudo apt install git
```

---

## 🛠️ 2. Configure Git with Your GitHub Account

Set your name and email (ensure the email matches your GitHub account):

```bash
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
```

Verify your configuration:

```bash
git config --global --list
```

---

## 🔐 3. Generate SSH Key

Create a new SSH key (recommended: `ed25519`):

```bash
ssh-keygen -t ed25519 -C "your-email@example.com"
```

- When prompted, press `Enter` to save in the default location (`~/.ssh/id_ed25519`)
- You may optionally enter a secure passphrase

Start the SSH agent and add your new key:

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

---

## 🧷 4. Add Your SSH Key to GitHub

Display the public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

Then:

1. Log in to [GitHub](https://github.com)
2. Go to **Settings** → **SSH and GPG keys**
3. Click **New SSH key**
4. Give it a name (e.g., "My Laptop"), paste the key, and save

---

## ✅ 5. Test the Connection

Check that your SSH key is working correctly:

```bash
ssh -T git@github.com
```

You should see a success message like:

> Hi `your-username`! You've successfully authenticated...

---

## 📥 6. Clone a GitHub Repository

Use SSH:

```bash
git clone git@github.com:username/repo-name.git
```

Or, if you didn’t set up SSH, use HTTPS:

```bash
git clone https://github.com/username/repo-name.git
```

---

## 🚀 7. Push Your Own Project to GitHub

To initialize and push an existing folder to GitHub:

```bash
cd your-project-folder
git init
git remote add origin git@github.com:username/repo-name.git
git add .
git commit -m "Initial commit"
git push -u origin master
```

---

## 🧠 Quick Tips

- Use `git status` often to check changes
- Use `git log` to see commit history
- Use `git branch` to manage feature branches

---


Happy coding!!! 💻✨