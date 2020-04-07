.. _citrixfiles:

-------------------------------
Nutanix Filesを利用したユーザーカスタマイズ
-------------------------------

:ref:`citrixnpdesktops` でご紹介したように、非永続デスクトップを展開する際の重要な課題は、ユースケースに基づいて適切なレベルのユーザーカスタマイズを引き続き提供できるようにすることです。 EUCスペースには複数のサードパーティソリューションが存在しますが、Citrixは、Citrix User Profile Management（UPM）やCitrix User Personalization Layers（UPL）など、すぐに使用できるオプションを複数提供しています。これらのソリューションはどちらも、プロファイルとレイヤーをホストするファイルサーバーを必要とします。

ユーザーのカスタマイズに加えて、ユーザーにはファイル用の外部ストレージも必要です。非永続デスクトップ上のリダイレクトされていないデスクトップフォルダーに保存するとデータが失われるためです。

Nutanix Filesは、EUC環境向けにネイティブの分散ファイルサービスを提供するソフトウェア定義のアプローチを導入しています。別のNASインフラストラクチャは必要ありません。Filesは、仮想デスクトップと同じインフラストラクチャに展開することもできます。

**このラボでは、Nutanix Filesを構成して、非永続的なCitrixデスクトップにユーザープロファイルとデータストレージを提供します。**

プロファイル共有の作成
+++++++++++++++++++++++

この演習では、時間とリソースを節約するために、Nutanix Filesインスタンスは既にクラスターにデプロイされています。 Nutanix Filesをデプロイする簡単なデモは `こちら <https://www.youtube.com/watch?v=gJagnILsd94>`_ で確認できます。

#. **Prism Element > File Server > Share/Export** と進み、 **+ Share/Export** をクリックする。

   .. figure:: images/1.png

#. **Basic** で、以下を入力し、 **Next** をクリック:

   - **Name** - *Initials*\ **-CitrixProfiles**
   - **Description** - User profiles and data
   - **File Server** - BootcampFS
   - **Select Protocol** - SMB

   .. note::

      この環境に用意されたファイルサーバはシングル構成のため、3台構成の際に提供される標準共有タイプと分散共有タイプのオプションは提供されません。 <プロファイルおよび分散共有に関する情報 : Filesクラスター内のすべてのFSVMにユーザーのホームディレクトリを均等に分散することにより、このユースケースのデータの共有を最適化します。>

#. 以下を入力し **Next > Create** をクリックする。:

   - **Enable Access Based Enumeration (ABE)** を選択
   - **Self Service Restore** を選択
   - **Blocked File Types** - .mp3、.mp4

   .. figure:: images/13.png

   .. note::

     Access Based Enumeration（ABE）はMicrosoft Windows（SMBプロトコル）機能であり、ユーザーはファイルサーバー上のコンテンツを閲覧するときに、読み取りアクセス権を持つファイルとフォルダーのみを表示できます。

     Self Service Restoreは、SMB共有での「以前のバージョン」機能を有効にしました。

     これらの機能はいずれも、先の手順で作成した共有ごとに有効化/無効化できます。

#. 上記の **Steps 1-3** を再度実施し、 *Initials*\ **-DepartmentShare** という共有名で作成する。 ※ただし、 **Blocked File Types** は空白で作成する。

   .. figure:: images/14.png

#. **Prism Element > File Server > File Server** と進み、 **BootcampFS** を選択し、**Protect** をクリック。

   .. figure:: images/2.png

     デフォルトのSelf Service Restore スケジュールを確認してください。この機能は、Windowsの「以前のバージョン」機能のスナップショットスケジュールを設定します。Windowsの「以前のバージョン」機能をサポートすることにより、エンドユーザーは、ストレージ管理者またはバックアップ管理者に依頼することなく、自身でファイルの変更をロールバックできます。また、これらのローカルスナップショットは、ローカル障害からファイルサーバークラスターを保護するだけではなく、ファイルサーバークラスター全体のレプリケーションをリモートNutanixクラスターに対して実行することもできます。

#. *Initials*\ **-WinTools** VM内のエクスプローラーから ``\\BootcampFS.ntnxlab.local\Initials-CitrixProfiles\`` にアクセスできることを確認する。

   .. figure:: images/3.png

   .. note::

     クォータ、アンチウイルス統合、監視などを含むファイル機能の詳細については、Nutanixポータルの `Nutanix Files Guide <https://portal.nutanix.com/#/page/docs/details?targetId=Files-v3_6:Files-v3_6>`_ を参照してください。

共有許可の設定
+++++++++++++++++++++++++++++

ファイルSMB共有のアクセス制御の管理は、Windowsを通じてネイティブに実行されます。この演習では、共有のアクセス許可を構成して、すべてのユーザーが共有内に所有するトップレベルのディレクトリを作成できるようにします。ユーザーが仮想デスクトップにログインすると、ADユーザー名に基づいて作成されたフォルダーが自動的に作成されます。

#. *Initials*\ **-WinTools** VMにてエクスプローラーを開き ``\\BootcampFS.ntnxlab.local\`` にアクセスする。

#. 作成した共有フォルダを右クリックし、 **Properties** を開く。

   .. figure:: images/4.png

#. Select the **Security** tab and click **Advanced**.

#. Select the default **Users (BootcampFS\\Users)** entry and click **Remove**.

#. Click **Add**.

#. Click **Select a principal** and specify **Everyone** in the **Object Name** field. Click **OK**.

#. Fill out the following fields and click **OK**:

   - **Type** - Allow
   - **Applies to** - This folder only
   - Select **Read & execute**
   - Select **List folder contents**
   - Select **Read**
   - Select **Write**

   .. figure:: images/5.png

#. Click **OK > OK > OK**.

   .. figure:: images/6.png

Configuring Citrix User Profile Management
++++++++++++++++++++++++++++++++++++++++++

UPM runs as a system service installed as part of the Virtual Delivery Agent within the virtual desktop or XenApp server. While similar to Microsoft Roaming Profiles, it offers key advantages such as faster logons by streaming the profile on-demand, administrative controls to limit profile size, and detailed logging.

In this exercise you will enable UPM through the Citrix Policy engine, similar to Microsoft Group Policy.

#. In **Citrix Studio > Policies**, right-click **Policies > Create Policy**.

   .. figure:: images/7.png

#. Select **Profile Management > Basic Settings** from the **All Settings** drop down menu. Optionally you can filter for only policies supported on **1912 Single-Session OS** from the **All Versions** drop down menu.

   .. figure:: images/8.png

#. Search for **Enable Profile management** and click **Select**. Select **Enabled** and click **OK**.

   .. figure:: images/9.png

#. Search for **Path to user store** and click **Select**. Select **Enabled** and specify ``\\BootcampFS\Initials-CitrixProfiles\%USERNAME%\!CTX_OSNAME!!CTX_OSBITNESS!`` as the path. Click **OK**.

   .. figure:: images/10.png

   .. note::

     The specified path will not only create unique top level directories within the share for each user, but will also create a platform specific subdirectory for their profile to avoid incompatability issues, such as trying to apply a Windows 10 user profile to a Windows 2012 session.

#. Click **Next**.

#. Click **Assign** to the right of **Delivery Group**.

#. Select your Non-Persistent Delivery Group from the **Delivery Group** drop down menu. Click **OK**.

   .. figure:: images/11.png

   .. note::

     Studio offers many different means of applying policies. Across a more diverse environment it may make sense to configure UPM settings based on OUs or Tags.

#. Click **Next**.

#. Provide a friendly **Policy name** (e.g. *Initials*\ **-UPM**) and select **Enable policy**. Review your configuration and click **Finish**.

   .. figure:: images/12.png

Testing Profiles and Folder Redirection
+++++++++++++++++++++++++++++++++++++++

#. From your *Initials*\ **ToolsVM**, open http://ddc.ntnxlab.local/Citrix/NTNXLABWeb, login as **NTNXLAB\\operator02** and connect to a **Pooled Windows 10 Desktop**.

#. Within your virtual desktop, make some simple changes such as adding files to your Documents folder. Note the hostname of the desktop to which you are connected.

   .. figure:: images/afsprofiles15.png

#. Open **PowerShell** and try to create a file with a blocked file type by executing the following command:

   .. code-block:: PowerShell

      New-Item \\BootcampFS\INITIALS-CitrixProfiles\operator02\Win10RS6x64\UPM_Profile\Documents\test.mp3

   Observe that creation of the new file is denied.

#. Sign out of the **Pooled** desktop. Do not just close the Citrix Workspace session as the desktop will not be re-provisioned.

#. Again, log in to Citrix StoreFront as **NTNXLAB\\operator02** and connect to a **Pooled Windows 10 Desktop**. Note that your files and settings persist across sessions, despite the underlying desktop being freshly provisioned every time you log in.

#. Open ``\\BootcampFS\Initials-CitrixProfiles\operator02`` in **File Explorer**. Drill down into the directory structure to find the data associated with your user profile.

#. Sign out of your virtual desktop. **Do not simply disconnect or close the Citrix Workspace App**.

#. Log in to Citrix StoreFront as **NTNXLAB\\operator01** and connect to a **Pooled Windows 10 Desktop**. Open ``\\BootcampFS\Initials-CitrixProfiles\`` and note that you don't see or have access to **operator02**'s profile directory. Disable **Access Based Enumeration (ABE)** in **Prism > File Server > Share/Export > home > Update** and try again.

#. (Optional) Create and save a text file in the **Documents** folder of your non-persistent virtual desktop. After ~1 hour, return to your virtual desktop, modify and save the document you previously created. Right-click the file and select **Restore previous versions**. Select an available previous version of the document and click **Open** to access the file.

.. figure:: images/afsprofiles16.png

(Optional) Using Files with Citrix User Personalization Layers
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

The Citrix UPL feature of Citrix Virtual Apps and Desktops (VAD) extends capabilities of non-persistent Windows 10 desktops by preserving data and locally user installed applications (UIA) across sessions.  Citrix UPL is the same technology as App Layering User Layers but is integrated into the Citrix Virtual Delivery Agent (VDA) and uses the Citrix policy engine.  Citrix UPL has all the features and functionality of User Layers (UL) without having to go through the entire App Layering process or having to deploy the Enterprise Layering Manager (ELM) virtual appliance.

.. note::

   All applications the user installs locally in the virtual desktop are supported in Citrix UPL, except for the following items:

   - Enterprise applications, such as Microsoft Office and Visual Studio
   - Applications that modify network stack or hardware, such as a VPN client
   - Applications that have boot level drivers, such as antivirus programs
   - Applications that have drivers that use the driver store, such as a printer driver

   Instead of having the user install the applications listed above locally in the virtual desktop as part of their UPL, install these applications in the master image.

   Any applications that attempt to add or edit local users or groups will not have the changes persist.  Instead add any required local users or groups to the master image.

   For full requirements and recommendations, see `Citrix Product Documentation on Citrix Virtual Apps and Desktops User Personalization Layer <https://docs.citrix.com/en-us/citrix-virtual-apps-desktops/install-configure/user-personalization-layer.html>`_.

#. Return to **Prism Element > File Server > Share/Export**, click **+ Share/Export**.

#. Under **Basic**, fill out the following fields and click **Next**:

   - **Name** - *Initials*\ **-CitrixUPL**
   - **Description** - Citrix UPL storage
   - **File Server** - BootcampFS
   - **Select Protocol** - SMB

#. Click **Next > Create**.

#. From your *Initials*\ **-WinTools** VM, open ``\\BootcampFS.ntnxlab.local\`` in File Explorer.

#. Open your *Initials*\ **-CitrixUPL** share and create a new directory named **Users**.

   .. figure:: images/15.png

   .. note::

      The folder name is hard coded in Citrix UPL and must be named **Users**.

#. Return to **Citrix Studio > Policies**. Right-click your **UPM** policy and select **Disable**.

   You will be applying your UPL policy to the same group of desktops.

#. Click **Create Policy**.

#. Specify **User Layer** in the **Search** field to filter for the required settings.

   .. figure:: images/16.png

#. Select **User Layer Repository Path** and specify the path to your *Initials*\ **-CitrixUPL** share. Do not include the **Users** folder in the path, this will be appended automatically. Click **OK**

   .. figure:: images/17.png

#. Select **User Layer Size in GB** and specify a value of **20** GB. Click **OK**.

   .. note:: The default value of 0 will configure 10GB UPL disks.

#. Click **Next**.

#. Click **Assign** to the right of **Delivery Group**.

#. Select your Non-Persistent Delivery Group from the **Delivery Group** drop down menu. Click **OK**.

   .. figure:: images/11.png

   .. note::

      Citrix UPL works with Pooled-Random and Pooled-Static Machine Catalogs. Citrix UPL does not support Pooled-Static Machine Catalogs with Citrix Personal vDisk (now deprecated) or dedicated, persistent machines that save changes to local disk.

#. Click **Next**.

#. Provide a friendly **Policy name** (e.g. *Initials*\ **-UPL**) and select **Enable policy**. Review your configuration and click **Finish**.

#. From your *Initials*\ **ToolsVM**, open http://ddc.ntnxlab.local/Citrix/NTNXLABWeb, login as **NTNXLAB\\operator03** and connect to a **Pooled Windows 10 Desktop**.

#. Open ``\\BootcampFS.ntnxlab.local\<Initials>-CitrixUPL\Users`` in File Explorer and note there is now a directory for your user containing a VHD with your personal desktop layer.

   .. figure:: images/18.png

#. Download and install **Mozilla Firefox** on your desktop. Launch Firefox and configure as your default browser.

#. Restart your virtual desktop.

#. After ~2 minutes, return to Citrix StoreFront and launch another **Pooled Windows 10 Desktop**. Observe that Firefox in still installed and configured as your default browser. Launch Firefox and note that the initial setup does not run again, as it has saved the settings from the previous session.

   .. figure:: images/19.png

#. Disconnect from your virtual desktop.

Takeaways
+++++++++

- Nutanix Files provides native files services suitable for storing user profile, data, and Citrix User Personalization Layer VHD files.

- Citrix User Personalization Layer is a simplified version of App Layering User Layers for non-persistent Provisioning and Machine Creation Services images.

- Nutanix Files can be deployed on the same Nutanix cluster as your Citrix virtual desktops, resulting in better utilization of storage capacity and eliminating additional storage silos.

- Supporting mixed workloads (e.g. virtual desktops and file services) is further enhanced by Nutanix's ability to mix different node configurations within a single cluster, such as:

  - Mixing storage heavy and compute heavy nodes
  - Expanding a cluster with Storage Only nodes to increase storage capacity without incurring additional virtualization licensing costs
  - Mixing different generations of hardware (e.g. NX-3460-G6 + NX-6235-G5)
  - Mixing all flash nodes with hybrid nodes
  - Mixing NVIDIA GPU nodes with non-GPU nodes
