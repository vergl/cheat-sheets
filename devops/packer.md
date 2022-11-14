# Packer

## Main commands

### Vadidate configuration
```bash
packer validate .
```

### Run configuration
```bash
packer build .
```

## Useful options
- *-var-file="config-file-name.pkrvars.hcl"* - file with credentials (if no *.auto.pkrvals.hcl* is present)
- *-on-error=ask* - Prevent auto-terminating and asks for manual action
- *-debug* - Asks 'Press enter to continue' on each atomic operation
- *PACKER_LOG=1 PACKER_LOG_PATH=filename ...* - enable logging

## Problems I faced

### Cannot eject ISO from cdrom drive
Packer 1.8.0 *Cannot eject ISO from cdrom drive*. GitHub [Issue](https://github.com/hashicorp/packer-plugin-proxmox/issues/102)
Current status: Fixed in Packer 1.8.1+

### QCOW2 is not supported
Packer 1.8.1+ (current version 1.8.4) can't create disks *format:qcow2*. The issues is described [here](https://github.com/hashicorp/packer-plugin-proxmox/issues/92). 
Current workaround: use *format:raw*.

