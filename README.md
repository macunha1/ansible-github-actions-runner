<h1 align="center">GitHub Actions self-hosted Runner Ansible role</h1>

An Ansible role that installs and configures GitHub Actions self-hosted Runners
inside one or multiple hosts, you can re-use it for many different URLs
(repositories or organizations) inside the same host in order to re-use it as
much as possible.

Main goals of this role:

- _avoid waste_: re-use the same host to provide a build environment for
  multiple repositories or organizations;
- _idempotence_: executing the role many times won't make anything break, steps
  have checks that validate whether or not they should be executed;

## Variables

For an exhaustive list of variables check the [defaults](defaults/main.yaml)
file. Ideally, all values will have commentaries describing what are their
purposes and by the default value you can tell the type.

### Required variables

Following values are required since there is no way to register the self-hosted
Runner without them

| Name                   | Description                                        |
| ---------------------- | -------------------------------------------------- |
| gh_runner_config_url   | GitHub Repository or Organization URL              |
| gh_runner_config_token | GitHub Registration token to authenticate the host |

## Example Playbook

Simplest use case: Single repository configuration on one host.

```yaml
- hosts: foo
  roles:
    - role: macunha1.github_actions_runner
      vars:
        gh_runner_config_labels:
          - linux
          - self-hosted

        gh_runner_config_url: https://github.com/macunha1/ansible-github-actions-runner
        gh_runner_config_token: AC5TNLJP9SBAFNEKKLLBLF264J8XO
```

Complex use case to which this role was created for

```yaml
- hosts: foo
  roles:
    - role: macunha1.github_actions_runner
      vars:
        gh_runner_config_labels:
          - linux
          - self-hosted

        gh_runner_config_url: https://github.com/macunha1/ansible-github-actions-runner
        gh_runner_config_token: AC5TNLJP9SBAFNEKKLLBLF264J8XO

    - role: macunha1.github_actions_runner
      vars:
        gh_runner_config_url: https://github.com/macunha1/another-repository
        gh_runner_config_token: AC5CQV3IJRR2OAFGEFCPJ0WJPJQXO

    - role: macunha1.github_actions_runner
      vars:
        gh_runner_config_url: https://github.com/macunha-acme-corp
        gh_runner_config_token: ACYWUR9MHGR9U58C34W9ZK00UNBF
```

Note that despite using the same host, each one of these GitHub Actions Runner
configuration will have its own path and credentials. Therefore, they can live
well in harmony without killing each other.

## Organization installs on one host

When you reuse the same host for multiple runners in the same Organization, set
`gh_runner_installation_path` to a unique folder for each role invocation.
That keeps the runner registrations isolated and avoids one runner replacing
another.

This is a workaround, not the preferred setup. Smaller dedicated VMs are the
better option when you can use them, because they keep environments segregated
and make runner state easier to reason about.

```yaml
- hosts: github-runners-ubuntu
  roles:
    - role: macunha1.github_actions_runner
      vars:
        gh_runner_installation_path: /usr/local/share/github-actions-runner/acme/runner1
        gh_runner_config_url: https://github.com/macunha-acme-corp
        gh_runner_config_token: ACYWUR9MHGR9U58C34W9ZK00UNBF
        gh_runner_config_name: acme-runner-1
        gh_runner_config_labels:
          - ubuntu22
          - linux
          - self-hosted

    - role: macunha1.github_actions_runner
      vars:
        gh_runner_installation_path: /usr/local/share/github-actions-runner/acme/runner2
        gh_runner_config_url: https://github.com/macunha-acme-corp
        gh_runner_config_token: ACYWUR9MHGR9U58C34W9ZK00UNBF
        gh_runner_config_name: acme-runner-2
        gh_runner_config_labels:
          - ubuntu22
          - linux
          - self-hosted

    - role: macunha1.github_actions_runner
      vars:
        gh_runner_installation_path: /usr/local/share/github-actions-runner/acme/runner3
        gh_runner_config_url: https://github.com/macunha-acme-corp
        gh_runner_config_token: ACYWUR9MHGR9U58C34W9ZK00UNBF
        gh_runner_config_name: acme-runner-3
        gh_runner_config_labels:
          - ubuntu22
          - linux
          - self-hosted
```

If you keep the same `gh_runner_installation_path` for all of them, the later
registration can replace the earlier one on the same org. Separate directories
avoid that overlap.

## Contribute

[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](http://makeapullrequest.com)

Feel free to fill [an issue](https://github.com/macunha1/ansible-github-actions-runner/issues)
containing feature request(s), or (even better) to send me a Pull request, I
would be happy to collaborate with you.

If this role didn't work for you, or if you found some bug during the execution,
let me know.
