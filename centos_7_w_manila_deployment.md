#Pre installation
- install centos7 minimal
- create stack user and give passwordless sudo powers

      sudo useradd -s /bin/bash -d /opt/stack -m stack
      echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack
     
- install git

      sudo yum install git-core -y
      
- install python3 and devel libraries
      
      sudo yum install python3 -y
      sudo yum install python3-devel

- Configure ntp to get correct time or ssl certificates will fail

      sudo yum install ntp -y
      sudo systemctl enable ntpd
      sudo systemctl start ntpd
        
- clone devstack 

      git clone https://git.openstack.org/openstack-dev/devstack
      cd devstask
- checkout the branch you want to use

      git checkout stable/stein
      
      
- Edit local.conf to use manila.

      cd /opt/devstack
      vi local.conf
      
  The below example disables horizon and manila-ui since we don't really need it

  contents of local.conf:
    
      [[local|localrc]]
      #If the repository has already been downloaded do not reclone
      RECLONE=False
      
      ADMIN_PASSWORD=secret
      # The database password gets set on first stack, so this should not
      # be changed between ./stack.sh and ./unstack.sh cycles.
      DATABASE_PASSWORD=$ADMIN_PASSWORD
      RABBIT_PASSWORD=$ADMIN_PASSWORD
      SERVICE_PASSWORD=$ADMIN_PASSWORD
      

      # This enables manila, you must have the plugin enable to be able 
      # to enable manila services using ENABLED_SERVICES      
      enable_plugin manila https://git.openstack.org/openstack/manila stable/stein

      # Note the "+=", it means add this to the list of services to enable.
      # if "=" then the default services are not enabled, because "=" overides
      # the contencts of ENABLED_SERVICES. 
      ENABLED_SERVICES+=manila,m-api,m-sch,m-shr,m-dat
      
      #disable_service adds the service to the disable list. Even if the service is a list
      # of ENABLED_SERVICES, it will be disabled.
      disable_service tempest
      disable_service horizon

      
- run stack.sh script

      ./stack.sh
      
- wait for a couple of hours, and check for status of stack:

      [stack@localhost devstack]$ source openrc admin
      WARNING: setting legacy OS_TENANT_NAME to support cli tools.
      [stack@localhost devstack]$ openstack service list 
      +----------------------------------+-------------+----------------+
      | ID                               | Name        | Type           |
      +----------------------------------+-------------+----------------+
      | 358047f0fe214e8784f19369ea01222b | manilav2    | sharev2        |
      | 658d70bcebd14cd6bcd15513f6dacdba | manila      | share          |
      | 9b1872829811487bb1cc49ff751efc8c | cinder      | block-storage  |
      | bb0742f10c8f45ee8fd54d6dacb0ea34 | glance      | image          |
      | bf3544daedf14ef1b8914b5f2e41bf57 | cinderv2    | volumev2       |
      | c90fd16d03354e11a4235633d1a20d28 | nova_legacy | compute_legacy |
      | d3de22dd964b4b62ba356f1f3aa3d017 | cinderv3    | volumev3       |
      | dd549fd1abbc411193347af9a8451080 | neutron     | network        |
      | e5cf2ab2d3364864b32a510274f381f3 | keystone    | identity       |
      | e8c41af920df4c5c8d9e6f58dde13262 | nova        | compute        |
      | f24eeb054d8942aab5f5aa5cfa80b627 | placement   | placement      |
      +----------------------------------+-------------+----------------+
      [stack@localhost devstack]$
      
