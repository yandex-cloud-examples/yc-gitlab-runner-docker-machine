# gitlab-runner-docker-machine

[Terraform](https://www.terraform.io/) manifest to dynamically create [GitLab Runner](https://docs.gitlab.com/runner/)-based GitLab CI workers, backed with a [docker-machine](https://gitlab.com/gitlab-org/ci-cd/docker-machine) executor and a [Yandex Cloud](https://github.com/yandex-cloud/docker-machine-driver-yandex) driver

## What for

This solution is fit for creating temporary GitLab Runner workers as requested by your CI to execute CI pipelines. You can use it to build and test `docker` containers, as well as for any other jobs that require a GitLab `docker` executor.

## Advantages 

* Regular GitLab Runner features.
* Does not require a Kubernetes cluster.
* Requires one persistent VM with minimum specs as follows: 2 CPU 50%, 2 GB RAM, 20 GB disk space, with temporary workers created on demand.
* Creating workers dynamically enables efficient use of computing resources.
* Due to short lifetime of a worker, you can fully benefit from NRDs, with little, if any, risk of losing your data.
* Isolation reduces information security risks for privileged `docker:dind` containers.
* You can flexibly manage worker lifetime with scheduling.

## How it works

The persistent VM with the above minimum requirements hosts your Gitlab Runner. When delegated a job from GitLab CI, it will create a virtual machine with the specified configuration (4 CPU, 8 GB RAM, 93 GB `network-ssd-nonreplicated` disk space), install `docker` on it, and run the job within Docker. Gitlab Runner can create multiple virtual machines at the same time to run multiple jobs concurrently. Once a job is complete, the Runner waits for new jobs to come within 30 minutes during business hours or 10 minutes the rest of the time; if no new jobs are delegated during the specified period, it deletes the temporary VM.  

## Employing a Terraform manifest

[Install Terraform and prepare your cloud](https://yandex.cloud/docs/tutorials/infrastructure-management/terraform-quickstart). 

To apply the manifest, specify the four required parameters: `cloud_id`, `folder_id`, `gitlab_url`, and `gitlab_registration_token`. You can set them in the `terraform.tfvars` file. 

> For other parameters, default values are as provided below:

```
# terraform.tfvars
cloud_id = "b1gxxxxx"
folder_id = "b1gxxxxx"
gitlab_registration_token = "GRxxxxxxx-UAxxxxxxx"
gitlab_url = "https://xxxxxx.gitlab.yandexcloud.net/"

```

Initialize Terraform: 

```
terraform init
```

Apply the manifest:

```
terraform apply
```

40-60 seconds after the manifest is successfully applied, a new runner will appear in the runners list. 

Note:
* The manifest creates a configuration file for GitLab Runner and a persistent VM. By default, the created VM has a public address.
* For the final configuration of your VM, run the `/root/postinstall.sh` script. To deliver the script and configuration file to the VM, use `user-data`. The script will install `gitlab-runner`, `docker-machiche`, and `docker-machine-yandex-driver`.
* To transfer the data for Gitlab Runner registration (URL and token), use [Yandex Lockbox](https://yandex.cloud/docs/lockbox/). The registration with GitLab CI takes place when the `gitlab-runner` service starts.
* To create temporary VMs, the service account linked to the persistent VM will be used.
* Security groups only allow access to temporary VMs from the persistent VM. 

## Software requirements

| Name | Version |
|------|---------|
| <a name="requirement_terraform"></a> [terraform](#requirement\_terraform) | >= 0.13 |

## Providers

| Name | Version |
|------|---------|
| <a name="provider_yandex"></a> [yandex](#provider\_yandex) | 0.85.0 |

## Modules

| Name | Source code | Version |
|------|--------|---------|
| <a name="module_network"></a> [network](#module\_network) | ./modules/network | n/a |

## Resources

| Name | Type |
|------|------|
| [yandex_compute_instance.gitlab_docker_machine](https://registry.tfpla.net/providers/yandex-cloud/yandex/latest/docs/resources/compute_instance) | resource |
| [yandex_iam_service_account.gitlab_docker_machine](https://registry.tfpla.net/providers/yandex-cloud/yandex/latest/docs/resources/iam_service_account) | resource |
| [yandex_kms_symmetric_key.gitlab_token_key](https://registry.tfpla.net/providers/yandex-cloud/yandex/latest/docs/resources/kms_symmetric_key) | resource |
| [yandex_kms_symmetric_key_iam_binding.gitlab_docker_machine_kms_roles](https://registry.tfpla.net/providers/yandex-cloud/yandex/latest/docs/resources/kms_symmetric_key_iam_binding) | resource |
| [yandex_lockbox_secret.gitlab_token](https://registry.tfpla.net/providers/yandex-cloud/yandex/latest/docs/resources/lockbox_secret) | resource |
| [yandex_lockbox_secret_version.gitlab_token_version](https://registry.tfpla.net/providers/yandex-cloud/yandex/latest/docs/resources/lockbox_secret_version) | resource |
| [yandex_resourcemanager_folder_iam_member.gitlab_docker_machine_roles](https://registry.tfpla.net/providers/yandex-cloud/yandex/latest/docs/resources/resourcemanager_folder_iam_member) | resource |
| [yandex_vpc_security_group.security_group_worker](https://registry.tfpla.net/providers/yandex-cloud/yandex/latest/docs/resources/vpc_security_group) | resource |
| [yandex_vpc_security_group.securtiy_group_master](https://registry.tfpla.net/providers/yandex-cloud/yandex/latest/docs/resources/vpc_security_group) | resource |
| [yandex_compute_image.ubuntu_lts](https://registry.tfpla.net/providers/yandex-cloud/yandex/latest/docs/data-sources/compute_image) | data source |

## Parameters

| Name | Description | Type | Default value | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_cloud_id"></a> [cloud\_id](#input\_cloud\_id) | cloud-id | `string` | n/a | yes |
| <a name="input_default_region"></a> [default\_region](#input\_default\_region) | Default Yandex Cloud region | `string` | `"ru-central1"` | no |
| <a name="input_default_zone"></a> [default\_zone](#input\_default\_zone) | Default availability zone | `string` | `"ru-central1-a"` | no |
| <a name="input_folder_id"></a> [folder\_id](#input\_folder\_id) | folder-id | `string` | n/a | yes |
| <a name="input_gitlab_registration_token"></a> [gitlab\_registration\_token](#input\_gitlab\_registration\_token) | gitlab registration token | `string` | n/a | yes |
| <a name="input_gitlab_url"></a> [gitlab\_url](#input\_gitlab\_url) | gitlab url | `string` | n/a | yes |
| <a name="input_gitlab_runner_tags"></a> [gitlab\_runner\_tags](#input\_gitlab\_runner\_tags) | gitlab runner tags | `string` | `""` | no |
| <a name="input_network_cidr"></a> [network\_cidr](#input\_network\_cidr) | network cidr | `string` | `"10.11.12.0/24"` | no |
| <a name="input_network_create"></a> [network\_create](#input\_network\_create) | create the network? | `bool` | `true` | no |
| <a name="input_network_description"></a> [network\_description](#input\_network\_description) | Network description | `string` | `"autocreated docker-machine network"` | no |
| <a name="input_network_id"></a> [network\_id](#input\_network\_id) | Existing network\_id(vpc-id) where resources will be created | `string` | `null` | no |
| <a name="input_network_name"></a> [network\_name](#input\_network\_name) | Network name | `string` | `"docker-machine"` | no |
| <a name="input_purpose"></a> [purpose](#input\_purpose) | Virtual machine purpose (prod, dev, stage, etc) | `string` | `"docker-machine"` | no |
| <a name="input_security_group_create"></a> [security\_group\_create](#input\_security\_group\_create) | create security group(s)? | `bool` | `true` | no |
| <a name="input_subnet_id"></a> [subnet\_id](#input\_subnet\_id) | Existing subnet id | `string` | `null` | no |
| <a name="input_user_pubkey_filename"></a> [user\_pubkey\_filename](#input\_user\_pubkey\_filename) | ssh public key filename | `string` | `"~/.ssh/id_rsa.pub"` | no |
| <a name="input_username"></a> [username](#input\_username) | Initialzation username | `string` | `"ubuntu"` | no |
| <a name="input_worker_runners_limit"></a> [worker\_runners\_limit](#input\_worker\_runners\_limit) | Maximum number of parallel workers | `string` | `"10"` | no |
| <a name="input_worker_cores"></a> [worker\_cores](#input\_worker\_cores) | yandex-cores | `string` | `"4"` | no |
| <a name="input_worker_disk_size"></a> [worker\_disk\_size](#input\_worker\_disk\_size) | yandex-disk-size | `string` | `"93"` | no |
| <a name="input_worker_disk_type"></a> [worker\_disk\_type](#input\_worker\_disk\_type) | yandex-disk-type | `string` | `"network-ssd-nonreplicated"` | no |
| <a name="input_worker_image_family"></a> [worker\_image\_family](#input\_worker\_image\_family) | yandex-image-family | `string` | `"ubuntu-2204-lts"` | no |
| <a name="input_worker_memory"></a> [worker\_memory](#input\_worker\_memory) | yandex-memory | `string` | `"8"` | no |
| <a name="input_worker_platform_id"></a> [worker\_platform\_id](#input\_worker\_platform\_id) | yandex-platform-id | `string` | `"standard-v3"` | no |
| <a name="input_worker_preemptible"></a> [worker\_preemptible](#input\_worker\_preemptible) | yandex-preemptible | `bool` | `true` | no |
| <a name="input_worker_use_internal_ip"></a> [worker\_use\_internal\_ip](#input\_worker\_use\_internal\_ip) | yandex-use-internal-ip | `bool` | `true` | no |

## Result

| Name | Description |
|------|-------------|
| <a name="output_docker-machine"></a> [docker-machine](#output\_docker-machine) | ssh command for connection |
