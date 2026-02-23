# cv-np-to-studios-migrations
## Purpose
Playbooks to download Network Provisioning Configlets and Container info and import into Static Configuration Studio.

There are three playbooks in the playbooks folder:
- **backup_network_provisioning_config.yml** - will connect to CV and download configlets and container information from Network Provisioning into a **cv_facts** folder.
    Three Files will be created in the cv_facts folder:
        **cvaas_configlets.yaml** - the configlets in YAML format
        **cvaas_containers.yaml** - information on the containers and the configlets attached to them
        **cvaas_devices.yaml** - information on the devices, the configlets attached to them, and the parent container
    A folder will be created called **cv_facts/configlets**.  Inside this folder, a file will be created with the name of each configlet, containing each configlet's data.
- **configure_static_config_studio_cv_deploy.yml** - Utilizes the AVD role cv_deploy and a static_config_manifest key to upload configlets, create "Containers", and attach the configlets to the appropriate containers in the Static Configuration Studio defined in the static_config_manifest.  It will create a workspace for this, but keep the workspace open.  This is because cv_deploy in static_config mode will not accept devices into the Inventory and Topology Studio.  This playbook requires a file called cvaas-service.tok created that contains a service account token from CV.

NOTE: This repo contains a devcontainer config that has AVD, and Ansible pre-installed.  If you do not wish to use the devcontainer, you will need to ensure that you have Ansible with the arista.cvp Ansible Galaxy Collection, as well as AVD.

## Instructions for use - backup_network_provisioning_config.yml
1. Create an inventory.yml file that contains the hosts to onboard to CVaaS, and a CVAAS group with a cvaas host in it.  See the example inventory.yml in the repo.
2. Log into CVaaS and create a service account and token.
3. Create a file called `cvaas-service.tok` and place the token inside it.
4. Run the playbook:
    `ansible-playbook -i <inventory_file> playbooks/backup_network_provisioning_config.yml`

The cv_facts folder and cv_facts/configlets folder will be created and populated.

## Instructions for use - configure_static_config_studio_cv_deploy.yml
1. Ensure you have an inventory file created that contains the FABRIC group and devices to add to the Static Configuration Studio (see example inventory.yml file)
2. Ensure you have a `cvaas-service.tok` file containing a CV Service Account token.
3. Edit the group_vars/STATIC_CONFIGS.yml file:
- Edit the configlets: key with a list of configlets to upload to the Static Configuration Studio.  Each configlet will have a `name:` key and a `file:` key that has the name of the configlet.  This will come out of the ./cv_facts/configlets folder.
- Edit the containters: key to create the container hierarchy in the Static Configuration Studio.  In each container or subcontainer, you will need to give a device_query: key that contains a list of all of the devices to add to the container or subcontainer (even if you will not be attaching a configlet to the container).  If applicable, add a configlets: key that contains a list of configlet names to attach.  The subcontainer: key will define a child container.  Finally, add subcontainer: keys for each device (with the appropriate device_query: string) that has a list of containers to attach to the device.  
- See [Static Configuration Studio Deployment](https://avd.arista.com/6.0/ansible_collections/arista/avd/roles/cv_deploy/index.html#static-configuration-studio-deployment) for more information.
5. Run the following playbook:
    `ansible-playbook -i <inventory_file> playbooks/configure_static_config_studio_cv_deploy.yml`
6. In CV, go into Provisioning -> Workspaces and select the workspace called "Static Configuration Studio via Ansible"
7. In that workspace, go into Inventory and Topology Studio and accept the devices to be onboarded into the Static Configuration Studio.
8. Go into the Static Configuration Studio and validate that the container structure was created, the devices are in the containers, and the configlets were uploaded to the configlet library and attached to the appropriate containers.
9. Build and submit the workspace.  This should result in no configuration change on the devices.
10. Go into Network provisioning and detach the configlets from the containers/devices. - I will post a playbook that will do this in the future.

NOTE: There are times when cv_deploy will not upload the configlets to the Static Configuration studio.  If that happens, abandon the workspace and run the playbook again.

