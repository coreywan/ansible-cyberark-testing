# Cyberark Ansible Testing

The goal I set out for this repo is to leverage CyberArk's vaulting capability to store credentials, and retrieve them with Ansible. Utilizing the two products together, I can automate with a better security posture. With CyberArk, I have the ability to audit and security manage priveleged credentials throughout my organization.  CyberArk also has capabilities that allows you to extend the vault usage to applications such as Ansible.


# Directions

**NOTE**: At the time of this initial discussion, there are some setup tasks that assume you are a WWT employee/contractor. 

1. Log into the WWT platform, and launch the [CyberArk Privileged Account Management](https://www.wwt.com/lab/cyberark-privileged-account-management) Lab.
2. Jump into the lab provided to you.
3. RDP from the jumphost over to `components.iam.com` using `Administrator` and the standard WWT password.
4. Download the required CyberArk packages from `\\10.255.16.16\atcdata\cyberark\Central Credential Provider-Rls-v10.10.3.zip`
5. Extract the ZIP somewhere, and open that in Windows Explorer.
6. Install the Central Credential Provider for Windows
   1. From your extracted file, launch the setup for CCP for Windows
      1. `.\Central Credential Provider-Rls-v10.10.3\Central Credential Provider\Central Credential Provider For Windows\setup.exe`
   2. As you run through the installation wizard, leave the defaults except:
      1. Vault's Connection Details:
         1. Address: 192.168.2.102
         2. Port: 1858
      2. Vault's Username and Password details:
         1. Username: Administrator
         2. Password: The WWT Standard
7. Install .NET Framework 3.5 from Server Manager
8. Install Central Credential Provider Web Service
   1. From your extracted file, launch the setup for CCP Web Service
      1. `.\Central Credential Provider-Rls-v10.10.3\Central Credential Provider\Central Credential Provider Web Service\setup.exe`
   2. Keep all the defaults
9. From the `jumpbox`, open Chrome and go to https://components.iam.com/PasswordVault/. Log in as `charles.xavier`.
10. Setup Permissions
    1.  Navigate to `Applications`, and add two applications.
        1.  AIMWebService, and allow it based on the user `IIS APPPOOL\DefaultAppPool` and path of `C:\inetpub\wwwroot\AIMWebService\bin\AIMWebService.dll` 
            1.  This will allow the AIMWebService we setup to talk with the Password Vault
        2.  Ansible, and allow it based on host address of `linuxserver.iam.com`
            1.  This will be the Application we use in Ansible to be able to authenticate and grab credentials
    2.  Naviage to `Policies` -> `Access Control (Safes)`
    3.  Click on `Linux`, and over on the right choose `Members` to update the members.
    4.  Add Three new members:
        1.  Prov_COMPONENTS
        2.  AIMWebService
        3.  Ansible
        4.  **NOTE**: When you add these new members to the Safe,  you should include `Use Accounts`, `Retrieve Accounts`, and `List Accounts`
11. Setup the Linux host:
    1.  ssh as `root` to `linuxserver` utilizing the standard WWT password.
    2.  Run the following to install some of the pre-req's for our automation:

        ```sh
        yum install -y epel-release
        yum install -y ansible git python2-pip
        pip install requests
        ansible-galaxy collection install cyberark.pas
        ```
    3. We should no longer need root, so let's jump out of it, and move to our normal user.  Saved in the putty configuration is our pre-setup session to authenticate as `demo_linux_01`
    4. Let's Clone this repo down.
        
        ```sh
        git clone https://github.com/coreywan/ansible-cyberark-testing.git
        cd ansible-cyberark-testing
        ```
    5. Execute the Ansible Playbook

        ```sh
        ansible-playbook ./pb_cyberark_query.yml
        ```