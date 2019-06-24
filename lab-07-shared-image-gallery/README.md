# Lab 7 - Shared Image Gallery

## Summary of Lab
In this exercise you will create a virtual machine using the shared image that you create. Initially, we will start by creating a custom image. Then, we will create shared image gallery and create shared image into this gallery. Finally, we will create a virtual machine using the shared image.\<br>

## Pre-Requisites
This Lab Exercise will assume that:\<br>
    You have added your Resource Group ID, Location, Source VM Name, Image Name, Shared Gallery Name, Shared Image Name, Shared Image Version and VM Name to the vars.yml file prior to executing this lab.\<br>

## Playbook 0 - Create Custom Image
    ansible-playbook 00-create-image.yml\<br>
    Resource Group Creation Task:\<br>
    ```
    - name: Create resource group if doesn't exist
      azure_rm_resourcegroup:
        name: "{{ resource_group }}"
        location: "{{ location }}"
    ```
    Virtual Network Creation Task:\<br>
    ```
    - name: Create virtual network
      azure_rm_virtualnetwork:
        resource_group: "{{ resource_group }}"
        name: "{{ virtual_network_name }}"
        address_prefixes: "10.0.0.0/16"
    - name: Add subnet
      azure_rm_subnet:
        resource_group: "{{ resource_group }}"
        name: "{{ subnet_name }}"
        address_prefix: "10.0.1.0/24"
        virtual_network: "{{ virtual_network_name }}"
    ```
    Public IP Address Creation Task:\<br>
    ```
    - name: Create public IP address
      azure_rm_publicipaddress:
        resource_group: "{{ resource_group }}"
        allocation_method: Static
        name: "{{ ip_name }}"
    ```
    Virtual Network Inteface Cards Creation Task:\<br>
    ```
    - name: Create virtual network inteface cards for VM A and B
      azure_rm_networkinterface:
        resource_group: "{{ resource_group }}"
        name: "{{ network_interface_name }}"
        virtual_network: "{{ virtual_network_name }}"
        subnet: "{{ subnet_name }}"
    ```
    Virtual Machine Creation Task:\<br>
    ```
    - name: Create VM
      azure_rm_virtualmachine:
        resource_group: "{{ resource_group }}"
        name: "{{ source_vm_name }}"
        admin_username: testuser
        admin_password: "Password1234!"
        vm_size: Standard_B1ms
        network_interfaces: "{{ network_interface_name }}"
        image:
          offer: UbuntuServer
          publisher: Canonical
          sku: 16.04-LTS
          version: latest
    - name: Generalize VM
      azure_rm_virtualmachine:
        resource_group: "{{ resource_group }}"
        name: "{{ source_vm_name }}"
        generalized: yes
    ```
    Custom Image Creation Task:\<br>
    In this task. we will use the azure_rm_image module. \<br>
    ```
    - name: Create custom image
      azure_rm_image:
        resource_group: "{{ resource_group }}"
        name: "{{ image_name }}"
        source: "{{ source_vm_name }}"
    ```
    After this playbook completes, you can visit the Azure Portal(https://portol.azure.com) and find the custom image in your resource group.\<br>

## Playbook 1 - Create Shared Image Gallery
    ansible-playbook 01-create-shared-image-gallery.yml\<br>
    Shared Image Gallery Creation Task:\<br>
    In this task, we will use the azure_rm_gallery module to create a shared image gallery. \<br>
    ```
    - name: Create shared image gallery
      azure_rm_gallery:
        resource_group: "{{ resource_group }}"
        name: "{{ shared_gallery_name }}"
        location: "{{ location }}"
        description: This is the gallery description.
    ```
    Once again, visit the Azure Portal(https://portol.azure.com) and in your resource group you will see a new shared image gallery.\<br>

## Playbook 2 - Create Shared Image
    ansible-playbook 02-create-image.yml\<br>
    Shared Image Creation Task:\<br>
    In this task, we will use the azure_rm_galleryimage module to create a shared image in your gallery. \<br>
    ```
    - name: Create shared image
      azure_rm_galleryimage:
        resource_group: "{{ resource_group }}"
        gallery_name: "{{ shared_gallery_name }}"
        name: "{{ shared_image_name }}"
        location: "{{ location }}"
        os_type: linux
        os_state: generalized
        identifier:
          publisher: myPublisherName
          offer: myOfferName
          sku: mySkuName
        description: Image Description  
    ```
    Gallery Image Version Creation Task: \<br>
    In this task, we will use the azure_rm_galleryimageversion module to create a simple gallery image version. \<br>
    ```
    - name: Create or update a simple gallery Image Version
      azure_rm_galleryimageversion:
        resource_group: "{{ resource_group }}"
        gallery_name: "{{ shared_gallery_name }}"
        gallery_image_name: "{{ shared_image_name }}"
        name: 10.1.3
        location: "{{ location }}"
        publishing_profile:
          end_of_life_date: "2020-10-01t00:00:00+00:00"
          exclude_from_latest: yes
          replica_count: 3
          storage_account_type: Standard_LRS
          target_regions:
            - name: West US
              regional_replica_count: 1
            - name: East US
              regional_replica_count: 2
              storage_account_type: Standard_ZRS
          managed_image:
            name: "{{ shared_image_name }}"
            resource_group: "{{ resource_group }}"
    ```
    After this playbook completes, you can visit the Azure Portal(https://portol.azure.com). Open your resource group, and you can find your shared image gallery. From the gallery you can see the shared image that you just created. There is the resource id of the shared image. Copy the resource id and add it to your vars.yml file as 'image_id'. \<br>


## Playbook 3 - Create Virtual Machine Using Shared Image
    ansible-playbook 03-create-vm-using-shared-image.yml\<br>
    Virtual Machine Creation Task:\<br>
    In this task, we will use the azure_rm_virtualmachine module to create a virtual machine using the shared image that we create in the pervious tasks.\<br>
    ```
    azure_rm_virtualmachine:
      resource_group: "{{ resource_group }}"
      name: "{{ vm_name }}"
      vm_size: Standard_DS1_v2
      admin_username: adminUser
      admin_password: PassWord01
      managed_disk_type: Standard_LRS
      image:
        id: "{{ image_id }}"
    ```
    After this playbook competes, you can visit the Azure Portal(https://portol.azure.com) and find the newly-created virtual machine in your resource group.\<br>