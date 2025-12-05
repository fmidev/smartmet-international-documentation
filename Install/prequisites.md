# SMartMet HCI Cluster prequisites


### Prequisites: Networking:

* Networking / subnets:
	* 1 subnet for "cluster management"
  		* Should be accessible from IT mgmt network and from the workstations and from the installation workspace as well.
		* Min. 32 IP addresses - mask of 24 f.e.
  		* This can be the same subnet that is used as a general server network in the site. If so please continue reserving IPs and configuring access as described in the next sections.
	* 1 isolated subnet for internal "cluster sync" data
  		* No access needed from anywhere else.
		* Min. 8 IP addresses - mask of 29 f.e.
	* 1 isolated subnet for internal "CEPH" data
  		* No access needed from anywhere else.
		* Min. 16 IP addresses - mask of 28 f.e.


* DNS / registering new IP addresses:
	* New Public IP address with port forwarding (ports 80, 443) pointing to kubernetes.[localsuffix] - this must be accessible from internal networks as well
	* Register (private) new IP address on "cluster management subnet" -> hostname: kubernetes.[localsuffix]
	* Register (private) new IP address on "cluster management subnet" -> hostname: share.[localsuffix] (if this is taken the maybe for example smartdata.[localsuffix])
   	* Register (private) new IP address on "cluster management subnet" -> hostname: [countrycode]-lom-1.[localsuffix]
	* Register (private) new IP address on "cluster management subnet" -> hostname: [countrycode]-lom-2.[localsuffix]
	* Register (private) new IP address on "cluster management subnet" -> hostname: [countrycode]-lom-3.[localsuffix]
	* Register (private) new IP address on "cluster management subnet" -> hostname: [countrycode]-pve-1.[localsuffix]
	* Register (private) new IP address on "cluster management subnet" -> hostname: [countrycode]-pve-2.[localsuffix]
	* Register (private) new IP address on "cluster management subnet" -> hostname: [countrycode]-pve-3.[localsuffix]
	* Register (private) new IP address on "cluster management subnet" -> hostname: [countrycode]-node-1.[localsuffix]
	* Register (private) new IP address on "cluster management subnet" -> hostname: [countrycode]-node-2.[localsuffix]
	* Register (private) new IP address on "cluster management subnet" -> hostname: [countrycode]-node-3.[localsuffix]
	* Register (private) new IP address on "cluster management subnet" -> hostname: [countrycode]-smartmet-analysis.[localsuffix]
	* Register (private) new IP address on "cluster management subnet" -> hostname: [countrycode]-smartmet-processing.[localsuffix]
	* Register (private) alias: traefik.[localsuffix] -> kubernetes.[localsuffix]
	* Register (private) alias: portainer.[localsuffix] -> kubernetes.[localsuffix]
	* Register (private) alias: geoweb.[localsuffix] -> kubernetes.[localsuffix]
	* Register (public) wildcard alias: *.apps.[localsuffix] -> Public IP address defined in third bullet
		* OR if wildcard alias is not possible then also need the following:
		* Register (public) alias: data.[localsuffix] -> Public IP address defined in third DNS bullet
		* Register (public) alias: alert.[localsuffix] -> Public IP address defined in third DNS bullet

```
Note:
[countrycode] should be two letter digit
[localsuffix] should be the domain name that is actually used at the destination site

For example processing server hostname should be something like this:

fi-smartmet-processing.fmi.fi
```

* Firewall:
	* Allow SMB mounts to the SmartMet workstations from: [countrycode]-smartmet-processing.[localsuffix]
	* Allow SMB mounts to the SmartMet workstations from: Kubernetes + HA nodes ([countrycode]-node-1.[localsuffix], [countrycode]-node-2.[localsuffix], [countrycode]-node-3.[localsuffix], kubernetes.[localsuffix], share.[localsuffix])
	* Allow 80 & 443 to kubernetes.[localsuffix]

* VPN (or other) remote access capability
	* Allow VPN clients to access all new internal IP addresses


### Prequisites: Hardware installation

* All the equipment installed into racks
* Power cabling done (including any possible UPS)
* SFP modules and necessary cabling on the receiving side (6 x LACP link, so 12 SFP modules)
	* 6 ports on the int "cluster subnet" and
	* 6 ports on the "CEPH subnet"
* RJ45 cabling on the receiving side (3 x LACP link, so 6 cables in total)
	* All ports on the "cluster management subnet"


### Prequisites: Preconfiguration of the hardware:

* Switch all RAID controllers to HBA mode.
	* For example: https://www.dell.com/support/manuals/fi-fi/poweredge-rc-h730/perc9ugpublication/switching-the-controller-to-hba-mode?guid=guid-1fcc87e1-d534-451a-9947-56f1175886c5&lang=en-us
* Enable BIOS/EFI virtualization options.
* Disable any Intel E-cores (if applicable).
* Configure LOM access (iLO, iDRAC, iPMi or similar)
	* Use the IP addresses ([countrycode]-lom-[1-3].[localsuffix]) made above
