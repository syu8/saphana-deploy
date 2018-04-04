saphana-deploy
==============

This role deploys SAP HANA 1.0 SPS 12 and above (HANA2) on a properly configured  RHEL 6.7 or 7.x system.

Requirements
------------

This role requires the system to be prepared with saphana-preconfigure role. Please make sure that the provided storage is sufficient with SAP requirements.


Role Variables
--------------

### Configuration for Instance deployment

This role uses the same variables as the saphana-preconfigure role.  It expects the installation  directory to be available at `hana_installdir`.

The following variables need to be set in the host_vars file. See [Best Practises](http://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html?highlight=host_var#group-and-host-variables) for more details.

Example host_vars file
----------------------
     ---
     deployment_instance: true

     instances:
        l01:
           id_user_sidadm: "30210"
           pw_user_sidadm: "Adm12356"
           hana_pw_system_user_clear: "System123"
           hana_components: "client,server"
           hana_system_type: "Master"
           id_group_shm: "30220"
           hana_instance_hostname: vm31
           hana_addhosts:
           hana_sid: L01
           hana_instance_number: 10
           hana_system_usage: custom


Example Playbook
----------------

Here is an example playbook that installs a complete server

    ---
    - hosts: hana
      remote_user: root

      vars:
              # subscribe-rhn role variables
              reg_activation_key: myregistration
              reg_organization_id: 123456

              repositories:
                      - rhel-7-server-rpms
                      - rhel-sap-hana-for-rhel-7-server-rpms

              # disk-init role variables
              disks:
                      /dev/vdc: vg00
                      /dev/vdb: vg00
              logvols:
                      hana_shared:
                              size: 24G
                              vol: vg00
                              mountpoint: /hana/shared
                      hana_data:
                              size: 24G
                              vol: vg00
                              mountpoint: /hana/data
                      hana_logs:
                              size: 12G
                              vol: vg00
                              mountpoint: /hana/logs
                      usr_sap:
                              size: 49G
                              vol: vg00
                              mountpoint: /usr/sap


              # rhel-system-roles.timesync variables
              ntp_servers:
                      - hostname: 0.rhel.pool.ntp.org
                        iburst: yes
                      - hostname: 1.rhel.pool.ntp.org
                        iburst: yes
                      - hostname: 2.rhel.pool.ntp.org
                        iburst: yes
                      - hostname: 3.rhel.pool.ntp.org
                        iburst: yes


              # SAP Precoonfigure role

              # SAP-Media Check
              install_nfs: "mynfsserver:/installi-export"
              installroot: /install
              hana_installdir: "{{ installroot + '/HANA2SPS02' }}"

              hana_pw_hostagent_ssl: "MyS3cret!"
              id_user_sapadm: "30200"
              id_group_shm: "30220"
              id_group_sapsys: "30200"
              pw_user_sapadm_clear: "MyS3cret!"

      roles:
              - { role: mk-ansible-roles.subscribe-rhn }
              - { role: mk-ansible-roles.disk-init }
              - { role: linux-system-roles.timesync }
              - { role: mk-ansible-roles.saphana-preconfigure }
              - { role: mk-ansible-roles.saphana-deployment }
              - { role: mk-ansible-roles.saphana-hsr }

License
-------

Apache License
Version 2.0, January 2004

Author Information
------------------

Markus Koch

Please leave comments in the github repo issue list
