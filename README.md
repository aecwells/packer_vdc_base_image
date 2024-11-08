# packer_vdc_base_image

To create a Packer configuration file that builds a Debian VM with support for `cloud-init` for use in VMware Cloud Director (VCD), follow this example. The Packer configuration will use the `vmware-iso` builder to create a VMware-compatible virtual machine.

```
{
  "variables": {
    "vm_name": "debian-cloud-init",
    "iso_url": "http://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-11.7.0-amd64-netinst.iso",
    "iso_checksum": "sha256:cb543ff8f936384479e17ffb22be99f42e2b6a4413b2f69e0136b3354d7d1ac1",
    "ssh_username": "packer",
    "ssh_password": "packer"
  },
  "builders": [
    {
      "type": "vmware-iso",
      "vm_name": "{{user `vm_name`}}",
      "iso_url": "{{user `iso_url`}}",
      "iso_checksum": "{{user `iso_checksum`}}",
      "iso_checksum_type": "sha256",
      "ssh_username": "{{user `ssh_username`}}",
      "ssh_password": "{{user `ssh_password`}}",
      "ssh_wait_timeout": "20m",
      "shutdown_command": "echo '{{user `ssh_password`}}' | sudo -S shutdown -P now",
      "tools_upload_flavor": "linux",
      "disk_size": 10000,
      "guest_os_type": "debian10-64",
      "headless": false,
      "http_directory": "http",
      "http_port_min": 8000,
      "http_port_max": 9000,
      "vmx_data": {
        "memsize": "1024",
        "numvcpus": "1",
        "scsi0.virtualDev": "lsilogic"
      }
    }
  ],
  "provisioners": [
    {
      "type": "shell",
      "inline": [
        "sudo apt-get update",
        "sudo apt-get install -y cloud-init open-vm-tools",
        "sudo bash -c 'echo "datasource_list: [ NoCloud, VMware, OVF ]" > /etc/cloud/cloud.cfg.d/99-vmware.cfg'",
        "sudo systemctl enable cloud-init"
      ]
    },
    {
      "type": "shell",
      "inline": [
        "sudo cloud-init clean",
        "sudo cloud-init init",
        "sudo cloud-init status"
      ]
    }
  ],
  "post-processors": [
    {
      "type": "vsphere",
      "cluster": "your-cluster",
      "host": "your-host",
      "datastore": "your-datastore",
      "username": "your-username",
      "password": "your-password",
      "insecure_connection": true,
      "vm_name": "{{user `vm_name`}}",
      "folder": "your-folder",
      "resource_pool": "your-resource-pool",
      "convert_to_template": true
    }
  ]
}
```

# Key sections explained:
1. Variables: 
  Define variables for VM name, ISO URL, checksum, and SSH credentials for use throughout the configuration.
2. Builders: 
  The `vmware-iso` builder is used to create a VMware-compatible VM from the Debian ISO.
  `guest_os_type`: For Debian, use `debian10-64` or another appropriate version.
  Basic hardware settings like disk size, memory, and CPU are set here.
3. Provisioners:
  Install `cloud-init` and `open-vm-tools` to ensure the VM can integrate with cloud environments (like VMware Cloud Director).
  Clean and initialize `cloud-init` to prepare the VM for use as a template in VCD.
4. Post-processors:
  The post-processor will upload the created VM to vSphere and convert it into a template ready for use in VMware Cloud Director.
  Update the `cluster`, `datastore`, `host`, `username`, `password`, etc., with your vSphere environment details.

# Next Steps:
1. Install Packer on your local machine.
2. Run the Packer template with packer build <template.json>.
3. Once the VM is built and uploaded to your vSphere environment, it will be available as a template for deployment in VMware Cloud Director.
