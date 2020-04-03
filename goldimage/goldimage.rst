.. _citrixgoldimage:

------------------------------------
ゴールドイメージの構築と最適化
------------------------------------

Windows Client OSは通常物理デバイス上（ラップトップPCなど）で動作することを前提としており、ハードウェアデバイスに直接接続され、また同ホスト上の負荷が高い仮想マシンに影響を受けるなどが発生しないことを想定しております。そのため、そのOSをVMとしてインストールすると想定外の動作をする可能性があり、仮想環境上で動作させるための最適化が必要です。

この演習では、Windows 10 VMにCitrix Virtual Desktop Agentをインストールし、Citrix OptimizerとVMware OS Optimization Toolの両方を使用してVMを最適化します。

ゴールドイメージ用VMの構築
++++++++++++++

#. Prism Centralで :fa:`bars` > **仮想インフラ（Virtual Infrastructure）** > **仮想マシン（VMs）** を選択。

   .. figure:: images/1.png

#. **仮想マシンを作成（Create VM）**をクリック。

#. 以下を入力する:

   - **Name** - *Initials*\ -GoldImage
   - **Description** - (オプション) WinToolsVM.
   - **vCPU(s)** - 2
   - **Number of Cores per vCPU** - 1
   - **Memory** - 4 GiB

   - **+ Add New Disk** をクリック
       - **Type（タイプ）** - DISK
       - **Operation（オペレーション）** - Clone from Image Service（イメージサービスからクローン）
       - **Image（イメージ）** - Win10v1903.qcow2
       - **Add** をクリック

   - **Add New NIC** をクリック
       - **VLAN Name** - Secondary
       - **Add** をクリック

#. **Save** をクリックしVMを作成する.

#. 作成したVMのチェックボックスをクリックし **Actions（アクション） > Power On（パワーオン）** .

.. _CtxPausingUpdates:

Windows Updatesの停止
+++++++++++++++

Windows 10ゴールドイメージの構築を開始する前に、Windows Updateが進行中でないことを確認することが重要です。Windows Updateによりクローン作成で問題が発生する可能性があります。

#. 作成したVMのチェックボックスをクリックし **Actions（アクション） > Launch Console（コンソールの起動）**

   - **User Name** - Nutanix
   - **Password** - nutanix/4u

#. **System Settings > Windows Update** を開き**Pause Updates for 7 Days** をクリックする。

   .. figure:: images/24.png

#. VMを再起動する。

VDAのインストール
++++++++++++++++++

Virtual Delivery Agent（VDA）は、ユーザー各物理マシンまたは仮想マシンに接続時に必要なドライバーとサービスのコレクションです。VDAにより、仮想デスクトップ用のマシンをDelivery Controllerに登録して、またDelivery Controllerがこれらのマシンをユーザーに割り当てることができます。VDAは、マシンとユーザーデバイス間のHDX接続、Citrixリモートプロトコルの確立、ライセンスの確認、Citrixポリシーの適用も担当します。

#. VMのコンソールで再起動完了を確認する。

#. **Computer Name** (e.g. *Initials*\ -GoldImage) を変更し以下の資格情報で**NTNXLAB.local** に参加する:
※検索ボックスで[ about ]と検索し、About画面下のRelated settingsの[System info]をクリックすることで該当画面に遷移できます。

   - **User Name** - NTNXLAB\\Administrator
   - **Password** - nutanix/4u

   .. figure:: images/1b.png

   .. note::

      Open **Control Panel > System and Security > System > Change Settings** to access the traditional Windows domain join field in Windows 10.

#. Restart your VM and log in using the following credentials:

   - **User Name** - NTNXLAB\\Administrator
   - **Password** - nutanix/4u

#. In **Prism Central**, select your GoldImage VM and click **Actions > Update**.

   .. figure:: images/2.png

#. Under **Disks > CD-ROM**, select :fa:`pencil` and fill out the following fields:

   - **Operation** - Clone from Image Service
   - **Image** - Citrix_Virtual_Apps_and_Desktops_7_1912.iso

#. Click **Update > Save**.

#. Within the VM console, open **D:\\AutoSelect.exe** to launch the Citrix installer.

   .. figure:: images/3.png

#. Select **Virtual Apps and Desktops > Start**.

   .. figure:: images/4.png

#. Select **Prepare Machines and Images** to begin installation of the Virtual Desktop Agent.

   .. figure:: images/5.png

#. Select **Create a MCS master image** and click **Next**.

   .. figure:: images/6.png

#. Under **Core Components**, select **Citrix Workspace App** in addition to the default **Virtual Desktop Agent** selection. Click **Next**.

   .. figure:: images/6b.png

#. Under **Additional Components**, select **Citrix User Personalization Layer** in addition to the default selections, and click **Next**.

   .. figure:: images/7.png

#. Under **Delivery Controller**, select **Let Machine Creation Services do it automatically** from the drop down, and click **Next**..

   .. figure:: images/8.png

# Under **Features**, click **Next**.

   .. figure:: images/9.png

#. Allow the installer to automatically configure required Windows Firewall port accessibility, click **Next**.

#. Click **Install** to begin the VDA installation. This process should take approximately 5 minutes.

#. When prompted, de-select **Collect diagnostic information** for Citrix Call Home and click **Next**.

   .. figure:: images/10.png

#. Click **Finish** and wait for the VM to restart.

Running Citrix Optimizer
++++++++++++++++++++++++

#. Within the VM console, download http://10.42.194.11/workshop_staging/CitrixOptimizer.zip and extract to a directory.

#. Right-click **CitrixOptimizer.exe** and select **Run as Administrator**.

   .. figure:: images/12.png

#. Select the recommended optimization template based on the Windows build being used for the gold image.

   .. figure:: images/13.png

#. Click **Select All** to select all available optimizations and click **Analyze**.

   .. figure:: images/14.png

#. Click **View Results** to see a detailed report of the status of each available optimization.

#. Return to the **Citrix Optimizer** and click **Done > Optimize** to apply the selected optimizations.

   .. figure:: images/15.png

#. Once the tool has completed, you can click **View Results** to view an updated report. You can now close the tool.

Running VMware OS Optimization Tool
+++++++++++++++++++++++++++++++++++

#. Within the VM console, download http://10.42.194.11/workshop_staging/VMwareOSOptimizationTool.zip and extract to a directory.

#. Right-click **VMwareOSOptimizationTool.exe** and select **Run as Administrator**.

#. Click the **Select All** checkbox. Scroll down to **Cleanup Jobs** and un-select the 4 available optimizations. Click **Analyze**.

   .. figure:: images/16.png

   .. note::

      The Cleanup Jobs are excluded from this exercise as they can be time consuming to apply.

#. Note the outstanding optimizations not applied in the **Analysis Summary** pane.

   .. figure:: images/17.png

#. Click **Optimize** to apply the remaining optimizations.

   .. figure:: images/18.png

#. Review the results and then restart your Gold Image VM.

Completing the Gold Image
+++++++++++++++++++++++++

XenDesktop provisions pools of desktops based on a hypervisor snapshot of the gold image. Unlike traditional hypervisors which can experience performance degradation from traversing long snapshot chains, Nutanix's redirect-on-write algorithm for implementing snapshots has no such drawback. This difference allows for flexibility in using gold image snapshots to maintain many gold image versions from a single VM. Watch `this video <https://youtu.be/uK5wWR44UYE>`_ for additional details on how Nutanix implements snapshots and cloning.

#. Once restarted, Perform a graceful shutdown of the VM from within the guest.

#. From **Prism Element**, take a snapshot of the VM (e.g. *Initials Post optimization and VDA install*)

   .. figure:: images/20.png

   .. note::

      This snapshot **must** be taken from Prism Element in order to be recognized by the Citrix AHV plug-in.

Takeaways
+++++++++

What are the key things learned in this exercise?

- Using MCS helps simplify the gold image by not having to manually specify (or depend on Active Directory to specify) what XenDesktop Delivery Controller(s) with which the image should attempt to register. This allows more flexibility in having a single gold image support multiple environments without external dependencies.

- EUC image optimization tools are not solution or hypervisor specific and can be easily applied to improve virtual desktop performance and increase host density.
