.. _citrixgettingstarted:

----------------------
Getting Started
----------------------

Welcome to the End User Computing bootcamp featuring Citrix Apps & Desktops. This bootcamp is meant to provide you with first hand experience in why Nutanix is an ideal platform for Citrix workloads. In addition to the benefits than Nutanix HCI brings to any virtual desktop deployment, such as linear scalability and consistent performance, Nutanix brings additional benefits that you'll explore through labs:

- Native tools for migrating existing desktop images from ESXi
- Citrix integration with AHV to provide a no-cost, easy to manage platform for desktop virtualization
- Fast desktop provisioning, including rolling out image updates to large pools of desktops
- Native file services with Nutanix Files to deliver user data, profiles, and User Personalization Layers
- Native microsegmentation with Nutanix Flow to secure a virtual desktop environment
- Rich monitoring and automation capabilities with Prism Ops


Deploying your Windows Tools VM
+++++++++++++++++++++++++++++++

#. In **Prism Central**, select :fa:`bars` **> Virtual Infrastructure > VMs**.

#. Click **Create VM**.

#. Select your assigned cluster and click **OK**.

#. Fill out the following fields:

   - **Name** - *Initials*-WinToolsVM
   - **Description** - (Optional) Description for your VM.
   - **vCPU(s)** - 2
   - **Number of Cores per vCPU** - 1
   - **Memory** - 4 GiB

   - Select **+ Add New Disk**
      - **Type** - DISK
      - **Operation** - Clone from Image Service
      - **Image** - WinToolsVM.qcow2
      - Select **Add**

   - Select **Add New NIC**
      - **VLAN Name** - Secondary
      - Select **Add**

#. Click **Save** to create the VM.

#. Select your VM and click **Actions > Power On**.

   Once booted, the VM will automatically complete the Sysprep process, join the **NTNXLAB.local** domain, and log in as the **NTNXLAB\\Administrator** user.
