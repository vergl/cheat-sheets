# Terraform

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

### Remove configuration (destroy VMs)
```bash
terraform destroy
```

## Useful options
- *--auto-approve* - disables prompt for approve
- *TF_REATTACH_PROVIDERS* - environment variable for debugging providers. Value for the variable is generated 
by the provider library when run in debug mode

## Problems I faced

### Occasional errors on run
Sometimes Proxmox returns lock errors or VM just get bugged. Looks like nothing you can do about it,
so destroy/apply the configuration one more time. 

Also it's a good idea to check VMs state after such errors because they don't get destroyed from time to time and
you have to do it manually. 

Unfortunately, this problem is quite common and makes using Terraform less convenient as it was supposed to be.

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