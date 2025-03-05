---
title: 'Advanced devcontainer configuration'
date: 2024-05-26T15:01:47Z
draft: false
tags: ["fedora","silverblue","VSCode","container"]
---

## An atomic linux distribution
I recently tried a new linux distribution : [Bluefin](https://projectbluefin.io/), it's an atomic operating system based on [Fedora Silverblue](https://fedoraproject.org/atomic-desktops/silverblue/). The atomic nature forces you to use containers when doing development, since your root filesystem is locked. I like it because it forces good practices,containerizing everithing so your system stay stable.

## Devcontainers
To help with the management of containers for application development Microsoft created the [Development Container](https://containers.dev/) specification. There is a visual studio code extension that enables you to easily manage your configurations.
## Adding extensions to the container environment
When VSCode remotes into a container or a machine the extensions needs to be installed on this remote. Whe can configure the DevContainer to have extensions preinstalled by setting `customizations.vscode.extensions`. Here is an example for my Dendron setup :

```json
{
	"name": "Dendron",
	"build": {
		"dockerfile": "Dockerfile"
	},
	"remoteUser": "node",
	"customizations": {
		"vscode": {
		"extensions": [
			"dendron.dendron",
			"dendron.dendron-markdown-shortcuts",
			"dendron.dendron-paste-image",
			"redhat.vscode-yaml"
			]
		}
	}
}
```

## Putting the configuration outside of the project's directory
By default the devcontainer configuration is placed in the project's directory, in `.devcontainer/devcontainer.json`. If you don't have control over the project this can be a problem as you need to avoid commiting this configuration in the repository. The devcontainer extension offers a solution : when creating a new devcontainer configuration with `ctrl+shift+p > Dev Containers: Reopen in Container` it asks you where you want to put the configuration, either in the "user data folder" or in the workspace (the project's directory). Selecting the firt option places the configuration in a folder in `~/.config/Code/User/globalStorage/ms-vscode-remote.remote-containers/configs/`.

If you want to use a custom dockerfile you can also place it in this folder and reference it in the configuration, the path is relative to the devcontainer.json file.

```json
{
	"name": "Rust",
	"build": {
		// Path is relative to the devcontainer.json file.
		"dockerfile": "Dockerfile"
	}
}
```
## Using FUSE in a Dev Container
FUSE (Filesystem in Userspace) allows a program to setup a virtual file system without adding a module to the Kernel. It doesn't work by defauld in a container and needs access to some ressources of the host system. You can easily configure the devcontainer to allow that :

```json
{
	"name": "Rust",
	"build": {
		"dockerfile": "Dockerfile"
	},
	"capAdd": [
		"SYS_ADMIN"
	],
	"runArgs": [
		"--security-opt",
		"apparmor:unconfined",
		"--device",
		"/dev/fuse"
	]
}
```

You also need to customize the container image to include the fuse executable, here's my Dockerfile for my rust project :

```Dockerfile
FROM mcr.microsoft.com/devcontainers/rust:1-bookworm
RUN apt update && apt upgrade -y
RUN apt install -y fuse3
```