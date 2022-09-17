---
id: m3tob8s1ecp0ldoc13s6lyo
title: User Setups
desc: ''
updated: 1663425391624
created: 1663423975058
---

## Adding username and email

After installation you need to configure user for git to use. Configuring git user used to identify author of a commit to setup a user you can use following command.

```bash
git config --global user.name "Your Name Here"
git config --global user.email "your@mail.here"
```

To view the setting use following command.

```bash
git config --global user.name
git config --global user.email
```

To configure a user for a specific repo you can use flag `–-local` instead of `–-global`.

Flag `--global` will indicate user used for all repository on the machine.

Flag `--local` will indicate user used for one repository or also you can ignore the flag all together

To check your git config use these following command.

```bash
git config --list
```

Personal configuration

```bash
git config --local user.name "Devtian Dicky"
git config --local user.email "33422734+pewpewtron@users.noreply.github.com"
```

## Modify username and email

To modify existing config either in local or global you can use edit as shown below

```bash
git config --local --edit
```

This will open text editor prompt to edit your configuration.

to modify specific value without prompting text editor use this command.

```bash
git config --global --replace-all user.name "Your New Name"
git config --global --replace-all user.email "Your new email"
```

## Delete username and email

You can use the --unset flag of git config to delete as shown below

```bash
git config --global --unset user.name
git config --global --unset user.email
```

If you have more variables for one config you can use

```bash
git config --global --unset-all user.name
```
