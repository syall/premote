# premote

## Overview

premote is a project remote CLI tool for managing AWS EC2 instances.

premote abstracts an EC2 instance under a project name. When using premote, the entire instance lifecycle is managed: creation, starting, stopping, and terminating.

By managing these events, the accruement of costs is kept to a minimum as long as there are no errors, but those fringe cases can usually be handled manually.

## Motivation

Instead of developing locally, premote connects the user to AWS EC2 instances where small-scale projects are essentially free. This provides several benefits:

- Taking advantage of consistent development environments with Amazon Machine Images
- Allowing low-spec devices (like Chromebooks) to act as developer machines

## Usage

### Installation

- Clone or Download the repository locally
- [Create a virtual environment](https://packaging.python.org/guides/installing-using-pip-and-virtual-environments/)
- Install the packages in `requirements.txt`

### Dependencies

premote is implemented in [Python](https://www.python.org/) and developed using Python 3.8.6, although older versions of Python 3 will probably work.

[Boto3](https://aws.amazon.com/sdk-for-python/) and its dependencies are the minimal dependencies needed to run premote, while the other dependencies in `requirements.txt` are only used in development.

### Storage

premote uses local storage in `~/.premote/` for individual project information, metadata for creating EC2 instances, and an EC2 private key.

```text
~/.premote/
├── project/ # Stores <project>.json files
├── meta.json
└── premote-keypair.pem
```

### AWS

Since premote uses Boto3, premote requires AWS credentials with programmatic access.

A simple guide to set up the credentials can be found in the [Boto3 Quickstart Page](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html#configuration).

For the Access Key Pair, the recommended minimal permissions is attaching the `AmazonEC2FullAccess` policy for:

- Creating an EC2 Key Pair
- Describing EC2 Instances
- Creating EC2 Instances
- Starting EC2 Instances
- Stopping EC2 Instances
- Terminating EC2 Instances

## Commands

### `init <project>`

Initializes a new Project with an EC2 Instance, then runs `start <project>` to SSH into the instance.

Notes:

- Creating the instance takes time until status is `running`
- Refer to [`start <project>`](#start-project) for further notes

### `start <project>`

Starts and SSHes into an existing Project EC2 Instance. The instance is stopped when the SSH session is exited properly.

Notes:

- Starting the instance takes time until reachable via SSH
- Use `logout` or `Ctrl+D` to properly exit an SSH session
- Stopping the instance takes time until status is `stopped`

### `delete <project>`

Deletes project and terminates the associated EC2 instance.

Note: Terminating the instance takes time until status is `terminated`

### `config`

Configures EC2 creation metadata by prompting for the:

- [Amazon Machine Image (AMI) ID](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html) (format: `ami-**`)
- [EC2 Security Group ID](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html) (format: `sg-**`)
- SSH Config File Path
- SSH User associated with the AMI

If no user input is provided, existing values are kept.

Note: The EC2 Security Group requires an inbound rule for SSH

## Technical Notes

### AWS EC2

- Every time an instance is started, a new Public DNS is assigned
- Every time an instance is stopped, its Public DNS is released
- Instance Status `running` is different than being reachable
- Instance IDs persist until terminated
- Instance Storage persists if the AMI has Elastic Block Storage (EBS)

### Python 3

- `argparse` for CLI arguments:
  - `subparsers = parser.add_subparsers()`
  - `subparsers.add_parser()`
  - `parser.set_defaults()`
  - `formatter_class=argparse.RawDescriptionHelpFormatter`
  - `metavar` compared to `dest`
  - `description` and `epilog`
- `os` for the file system:
  - `os.path.exists`
  - `os.path.isfile`
  - `os.path.isdir`
  - `os.path.expanduser`
  - `os.system`
- Boto3, the AWS SDK for Python:
  - `client`, `resource`, `waiter`
  - Documentation for EC2 is huge and laggy
- Miscellaneous
  - `with` does not create a block scope
  - `print('error msg', file=sys.stderr)`
  - Iterate `dicts` with `.keys()`, `.values()`, `.items()`
  - Using `**kwargs` and `\` to write shorter lines
  - `sys.exit()` for exiting an application
  - Utility classes with `@staticmethod`
