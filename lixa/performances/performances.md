# LIXA Performances

LIXA provides a test tool that can be used to measure performances: lixat.
All the measures reported in this section have been collected using a couple of
VMs in Azure public cloud West Europe region.
The first virtual machine is used to host the LIXA state server (lixad), the second virtual machine is used to host the LIXA client test program (lixat).
Both the virtual machines are configured as below:
- Operating system: Linux Ubuntu 18.04-LTS
- VM generation: V1
- Size: Standard D2s v3
- vCPUS: 2
- RAM: 8 GiB
- Azure disk encryption: Enabled
- Disk Size: 30 GiB
- Disk Storage account: Premium SSD
- Disk Encryption: SSE with PMK
- Disk Host caching: Read/write

All the above are default parameters for D2s v3 on 2020Q2
