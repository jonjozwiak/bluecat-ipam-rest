Role Name
=========

This role works with Bluecat Address Manager (Proteus) REST API to acquire and release IP addresses and their associated DNS names.  It does a quick deploy rather than full deploy.  It takes a target hostname as input and sets the following variables while acquiring and IP which can be used in subsequent plays: 

    ipAddress
    ipNetmask
    ipGateway

This role was tested with Bluecat Address Manager 8.1.0

Requirements
------------

This role uses the 'ipaddr' filter to return the subnet mask for the given CIDR.  This requires the python netaddr module (from python-netaddr / python3-netaddr RPM).  Alternatively you could remove the ipNetmask section from the role.


Role Variables
--------------

Variables can be set overall in the top section of a playbook or in the include_role section as shown in the playbook example.  variables in the top section override those in the tasks section.  

    # Bluecat Address Manager Credentials of User with API access 
    bluecat_username: "apiuser"
    bluecat_password: "apipassword"
    bluecat_url: "https://bcn_proteus.example.com"
    
    # Configuration and view details from Bluecat
    bluecat_configuration_name: "Example"
    bluecat_dns_view: "internal"
    
    # Properties to pass through to the bluecat API acquire IP call (only one can be defined)
    address_properties: "offset=192.168.30.15"          # Start from this address
    address_properties: "skip=192.168.30.1-192.168.30.15"        # Skip these addresses
    address_properties: "|excludeDHCPRange=true"        # Skip DHCP range
    address_properties: "skip=10.10.10.128-10.10.11.200,10.10.11.210|offset=10.10.10.100|excludeDHCPRange=true|" # All in one
    
    # Choose no if using self signed certs.  yes if certs are valid
    validate_certs: "no"
    
    # Determine whether to acquire or release an IP/DNS name.  Default is present.  Options are:
    #    Create or Acquire IP/DNS:  present, acquire 
    #    Lookup IP by hostname: 	get, lookup
    #    Release IP/DNS:		absent, release
    state: "present"
    
    # Host name to acquire or release 
    target_hostname: "host.example.com"
    
    # The CIDR of the network to acquire an IP
    bluecat_network_cidr: "192.168.30.0/24"


Dependencies
------------


Example Playbook
----------------
NOTE- vars up top override include_role vars below.  If you are setting a variable in the include_role in tasks, don't state it in the vars at the top

    - name: Deploy and retire IP addresses
      hosts: localhost
      vars:
        bluecat_username: "apiuser"
        bluecat_password: "apipassword"
        bluecat_url: "https://bcn_proteus.example.com"
        bluecat_configuration_name: "Example"
        bluecat_network_cidr: "192.168.30.0/24"     # The CIDR of the network to acquire an IP
        bluecat_dns_view: "internal"
        address_properties: "offset=192.168.30.15"          # Start from this address
        #address_properties: "skip=192.168.30.1-192.168.30.15"        # Skip these addresses
        #address_properties: "|excludeDHCPRange=true"        # Skip DHCP range
        validate_certs: "no"        # Choose no if using self signed certs.. 
      gather_facts: false
      tasks:
        - name: Get IP Address 
          include_role: 
            name: jonjozwiak.bluecat-ipam-rest 
          vars: 
            target_hostname: "ansibletest.example.com"
    
        - name: Do something with the IP address returned 
          debug: var=ipAddress
    
        - name: Get IP Address
          include_role:
            name: jonjozwiak.bluecat-ipam-rest
          vars:
            target_hostname: "ansibletest2.example.com"
    
        - name: Do something with the IP address returned 
          debug: msg="IP address: <{{ipAddress}}>.  Netmask: <{{ipNetmask}}>. Gateway: <{{ipGateway}}>."
    
        - name: Release IP Address 
          include_role:
            name: jonjozwiak.bluecat-ipam-rest
          vars:
            target_hostname: "ansibletest.example.com"
            state: absent
    
        - name: Release IP Address 
          include_role:
            name: jonjozwiak.bluecat-ipam-rest
          vars:
            target_hostname: "ansibletest2.example.com"
            state: absent



License
-------

GPLv3

Author Information
------------------

Jon Jozwiak
