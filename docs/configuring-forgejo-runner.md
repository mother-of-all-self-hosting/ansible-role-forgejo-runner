<!--
SPDX-FileCopyrightText: 2020 Aaron Raimist
SPDX-FileCopyrightText: 2020 Chris van Dijk
SPDX-FileCopyrightText: 2020 Dominik Zajac
SPDX-FileCopyrightText: 2020 Mickaël Cornière
SPDX-FileCopyrightText: 2020-2024 MDAD project contributors
SPDX-FileCopyrightText: 2020-2024 Slavi Pantaleev
SPDX-FileCopyrightText: 2022 François Darveau
SPDX-FileCopyrightText: 2022 Julian Foad
SPDX-FileCopyrightText: 2022 Warren Bailey
SPDX-FileCopyrightText: 2023 Antonis Christofides
SPDX-FileCopyrightText: 2023 Felix Stupp
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024-2026 Suguru Hirahara

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up Forgejo Runner

This is an [Ansible](https://www.ansible.com/) role which installs [Forgejo Runner](https://code.forgejo.org/forgejo/runner) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

Forgejo Runner is a runner to use with [Forgejo Actions](https://forgejo.org/docs/latest/admin/actions/). It provides a way to perform CI using Forgejo.

See the project's [documentation](https://forgejo.org/docs/latest/admin/actions/runner-installation/) to learn what Forgejo Runner does and why it might be useful to you.

> [!WARNING]
> The projects' documentation does **not recommend** running Forgejo Runner on the same machine as the Forgejo instance for security reasons.

## Prerequisites

### Retrieve a registration token

To set up Forgejo Runner on a Forgejo instance, you will need to retrieve the registration token which is used for registering the runner.

The registration token can be obtained via Forgejo's web interface by going to `Site Administration -> Actions -> Runners -> Create new runner`. Refer to [this section](https://forgejo.org/docs/latest/admin/actions/runner-installation/#standard-registration) on the official documentation for the latest information.

## Adjusting the playbook configuration

To enable Forgejo Runner with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# forgejo_runner                                                       #
#                                                                      #
########################################################################

forgejo_runner_enabled: true

########################################################################
#                                                                      #
# /forgejo_runner                                                      #
#                                                                      #
########################################################################
```

### Set the Forgejo instance URL

It is necessary to specify the URL of the Forgejo instance as well, for which the runner is used. Add the following configuration to your `vars.yml` file:

```yaml
forgejo_runner_instance_url: "https://example.com"
```

If the Forgejo instance is managed by [ansible-role-forgejo](https://github.com/mother-of-all-self-hosting/ansible-role-forgejo), you can set the URL as below:

```yaml
forgejo_runner_instance_url: "{{ forgejo_hostname }}{{ forgejo_path_prefix if forgejo_path_prefix != '/' }}"
```

### Set the registration token

You also need to set the registration token retrieved on the Forgejo instance by adding the following configuration to your `vars.yml` file:

```yaml
forgejo_runner_registration_token: REGISTRATION_TOKEN_HERE
```

### Set runner's labels

It is required to specify the labels of a runner, which are used to determine which jobs the runner can run, and how to run them.

For example, you can specify a label to `forgejo_runner_labels` as below:

```yaml
forgejo_runner_labels:
  - ubuntu-22.04:docker://node:20-bullseye
```

Since the labels are an important aspect of the runner, they should be carefully chosen. Read [the official documentation](https://forgejo.org/docs/latest/admin/actions/configuration/#choosing-labels) for more information.

### Set the runner's name

It is also necessary to set the runner's name by adding the following configuration to your `vars.yml` file:

```yaml
forgejo_runner_runner_name: YOUR_RUNNER_NAME_HERE
```

### Increasing the capacity (optional)

By default the role specifies the capacity of the runner (how many concurrent tasks it can run) to `1`. You can increase it per the computation power of the machine where the runner is used by adding the following configuration to your `vars.yml` file:

```yaml
forgejo_runner_capacity: 2
```

### Extending the configuration

There are some additional things you may wish to configure about the service.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `forgejo_runner_environment_variables_additional_variables` variable

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, Forgejo Runner becomes available.

>[!NOTE]
> The runner will register with the Forgejo instance (provided via the `forgejo_runner_instance_url` variable) and generate a `.runner` file inside its configuration path. This file should not be modified manually. If for some reason you wish to force the registration to run again, you can delete the `.runner` file and restart the service.
>
> If you wish to change the labels associated with the runner, you can simply modify the `forgejo_runner_labels` variable and run the playbook again. There is no need to delete the `.runner` file and run the registration again.

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu forgejo-runner` (or how you/your playbook named the service, e.g. `mash-forgejo-runner`).
