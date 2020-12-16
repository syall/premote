# premote

## Overview

premote is a project remote CLI tool for managing AWS EC2 instances.

premote abstracts an EC2 instance under a project name. When using premote, the entire instance lifecycle is managed: creation, starting, stopping, and terminating.

By managing these events, the accruement of costs is kept to a minimum as long as there are no errors, but those fringe cases can usually be handled manually.

## Usage

### Installation

- Clone or Download the repository locally
- [Create a virtual environment](https://packaging.python.org/guides/installing-using-pip-and-virtual-environments/)
- Install the packages in `requirements.txt`

### Dependencies

premote is implemented in [Python](https://www.python.org/) and developed using Python 3.8.6, although older versions of Python 3 will probably work since it only depends on the `sys`, `os`, `json`, and `time` from the standard library.

[Boto3](https://aws.amazon.com/sdk-for-python/) and its dependencies are the minimal dependencies needed to run premote, while the other dependencies in `requirements.txt` are only used in development.

### AWS Credentials

Since premote uses Boto3, premote requires AWS credentials to access an AWS account programmatically.

A simple guide to set up the credentials can be found in the [Boto3 Quickstart Page](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html#configuration).

For the Access Key Pair, the recommended minimal permissions is attaching the `AmazonEC2FullAccess` policy for:

- Creating an EC2 Key Pair
- Describing EC2 Instances
- Creating EC2 Instances
- Starting EC2 Instances
- Stopping EC2 Instances
- Terminating EC2 Instances

### Storage

premote uses local storage in `~/.premote/` for individual project information, metadata for creating EC2 instances, and an EC2 private key.

```shell
~/.premote/
├── project/ # Stores <project-name>.json
├── meta.json
└── premote-keypair.pem
```

## Commands

### `init <project-name>`

`init` creates a project based on `<project-name>` by creating and saving information about an EC2 instance, then runs `start` to SSH into the EC2 instance.

- Creating an instance takes time until the instance status is `running`.
- Refer to [`start <project-name>`](#start-project-name) for further notes.

### `start <project-name>`

`start` starts and SSHes into the EC2 instance associated with the project `<project-name>`. When the SSH session is exited properly, the EC2 instance is stopped.

- Starting the instance takes time until it is reachable via SSH.
- Use the `logout` command or `Ctrl+D` shortcut to properly exit an SSH session.
- Stopping the instance takes time until the instance status is `stopped`.

### `delete <project-name>`

`delete` terminates the EC2 instance and deletes the local data associated with the project `<project-name>`.

- Terminating the instance takes time until the instance status is `terminated`.

### `config`

`config` prompts the user for a [Amazon Machine Image (AMI) ID](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html) (format: `ami-**`) and an [EC2 Security Group ID](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html) (format: `sg-**`). If no input is provided, the existing values will be used.

Currently, premote only supports Ubuntu AMIs (recommended 20 or 18 LTS). Other AMIs can be [found](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/finding-an-ami.html), but the SSH command will have to be modified in `premote` line 193. However, it is recommended to either find or build a base AMI that is generalized for most projects so that the configuration does not constantly change.

The selected EC2 Security Group minimally requires an inbound rule for SSH to connect to, but otherwise can be completely customizable:

Type | Protocol | Port range | Source
-----|----------|------------|----------
SSH  | TCP      | 22         | 0.0.0.0/0

### `help`

`help` will print out the usage information of premote:

```text
Usage: premote command [argument]
Commands:
- init   <project-name>
- start  <project-name>
- delete <project-name>
- config
- help
```

## Motivation

Instead of developing locally, premote connects the user to AWS EC2 instances where small-scale projects are essentially free. This provides several benefits:

- Taking advantage of consistent development environments with Amazon Machine Images
- Allowing low-spec devices (like Chromebooks) to act as developer machines
