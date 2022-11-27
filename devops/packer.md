# Packer
Packer is a free and open source tool for creating golden images for multiple platforms from a single source configuration.

## Prerequisites
### Install Packer CLI
Documentation: [Install Packer](https://developer.hashicorp.com/packer/tutorials/docker-get-started/get-started-install-cli)

Installation on MacOS with *brew*:
```bash
brew tap hashicorp/tap
brew install hashicorp/tap/packer
```
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
- *mkpasswd --method=SHA-512 --rounds=4096* - command to generate password's hash which is used in *user-data*. Vanilla Ubuntu does not have command *mkpasswd* installed so you have to install package whois beforehand (*mkpasswd* is a part of *whois*)

## Problems I faced

### Cannot eject ISO from cdrom drive
Packer 1.8.0 *Cannot eject ISO from cdrom drive*. GitHub [Issue](https://github.com/hashicorp/packer-plugin-proxmox/issues/102)
Current status: Fixed in Packer 1.8.1+

### QCOW2 is not supported
Packer 1.8.1+ (current version 1.8.4) can't create disks *format:qcow2*. The issues is described [here](https://github.com/hashicorp/packer-plugin-proxmox/issues/92). 
Current workaround: use *format:raw*.

