# Terraform
Terraform is an open-source infrastructure as code software tool that enables you to safely and predictably create, change, and improve infrastructure.

## Prerequisites
### Install Terraform CLI
Documentation: [Install Terraform](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli)

Installation on MacOS with *brew*:
```bash
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
```

## Main commands

### Initialize Terraform
```bash
terraform init
```

### Test configuration
```bash
terraform plan
```

### Run configuration (create VMs)
```bash
terraform apply
```


### Run specific configuration (create VMs)
```bash
terraform apply -target=resource_type.resource_name
```

### Remove configuration (destroy VMs)
```bash
terraform destroy
```

## Useful options
- *--auto-approve* - Disables prompt for approve
- *TF_REATTACH_PROVIDERS* - Environment variable for debugging providers. Value for the variable is generated 
by the provider library when run in debug mode
- *-parallelism=n* - Limit the number of concurrent operation as Terraform walks the graph.

## Problems I faced

### Occasional errors on run
Sometimes Proxmox returns lock errors or VM just get bugged. Looks like nothing you can do about it,
so destroy/apply the configuration one more time. 

Also it's a good idea to check VMs state after such errors because they don't get destroyed from time to time and
you have to do it manually. 

Unfortunately, this problem is quite common and makes using Terraform less convenient as it was supposed to be.

*Workaround:* set `-parallelism=1` to decrease the chance of locks and other problems.

### VMs don't get IP address from cloud-init configuration
There is a [Github Issue](https://github.com/Telmate/terraform-provider-proxmox/issues/603) about it. After debugging
*terraform-provider-proxmox* and *proxmox-api-go* I realized that it is the problem on the side of Proxmox itself (tested on 7.2 and 7.3, problem still exists).

VM gets IP address from DHCP on the first boot, even though it was set as static in `ipconfig0` in cloud-init settings.  Nevertheless this VM gets static IP after reboot, so workaround I found was **remote-exec** provider which reboots the machine manually after it was created.

```terraform
  provisioner "remote-exec" {
    inline = [
      "sudo shutdown -r +0"
    ]
    connection {
      type     = "ssh"
      host     = self.ssh_host
      user     = "${var.ssh_user}"
      password = "${var.ssh_password}"
      port     = self.ssh_port
    }
  }
```