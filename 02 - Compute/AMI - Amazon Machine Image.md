---
tags:
  - aws/ec2
  - compute
category: Compute
---

# AMI - Amazon Machine Image

> [!INFO] Definition
> An AMI is a template that contains the software configuration (operating system, application server, and applications) required to launch an instance.

## Key Facts
* **Regional**: AMIs are stored in a specific region. To use an AMI in another region, you must **copy** it.
* **Permissions**:
	* **Public**: Available to all AWS accounts.
	* **Explicit**: Shared with specific AWS accounts.
	* **Implicit (Owner)**: Only available to the owner.

## AMI Components
* **Root Volume Template**: (EBS snapshot or Instance Store template).
* **Block Device Mapping**: Defines which volumes to attach when the instance is launched.
* **Launch Permissions**: Who can use the AMI.

## Custom AMIs
* You can "Bake" an AMI by launching an instance, configuring it, and then creating an image from it to save time during scaling.