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

#. **仮想マシンを作成（Create VM）** をクリック。

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

#. **System Settings > Windows Update** を開き **Pause Updates for 7 Days** をクリックする。

   .. figure:: images/24.png

#. VMを再起動する。

VDAのインストール
++++++++++++++++++

Virtual Delivery Agent（VDA）は、ユーザー各物理マシンまたは仮想マシンに接続時に必要なドライバーとサービスのコレクションです。VDAにより、仮想デスクトップ用のマシンをDelivery Controllerに登録して、またDelivery Controllerがこれらのマシンをユーザーに割り当てることができます。VDAは、マシンとユーザーデバイス間のHDX接続、Citrixリモートプロトコルの確立、ライセンスの確認、Citrixポリシーの適用も担当します。

#. VMのコンソールで再起動完了を確認する。

#. **Computer Name** (e.g. *Initials*\ -GoldImage) を変更し以下の資格情報で**NTNXLAB.local** に参加する:

   - **User Name** - NTNXLAB\\Administrator
   - **Password** - nutanix/4u

   .. figure:: images/1b.png

   .. note::

       **Control Panel > System and Security > System > Change Settings** とアクセスすることで上記画面に遷移できます。

#. VMを再起動し、以下の資格情報でログイン。:

   - **User Name** - NTNXLAB\\Administrator
   - **Password** - nutanix/4u

#. **Prism Central** でゴールドイメージVM のチェックボックスを選択し **Actions（アクション） > Update（更新）** をクリック。

   .. figure:: images/2.png

#. **Disks > CD-ROM**　を選択し :fa:`pencil` をクリックし以下を入力:

   - **Operation（オペレーション）** - Clone from Image Service（イメージサービスからクローン）
   - **Image（イメージ）** - Citrix_Virtual_Apps_and_Desktops_7_1912.iso

#. **Update > Save**をクリック。

#. ゴールドイメージVMのコンソールにて **D:\\AutoSelect.exe** を開きCitrix installerを起動。

   .. figure:: images/3.png

#. **Virtual Apps and Desktops > Start** を選択

   .. figure:: images/4.png

#. **Prepare Machines and Images** を選択し、Virtual Desktop Agentのインストールを開始する。

   .. figure:: images/5.png

#. **Create a MCS master image** を選択し **Next** をクリック。

   .. figure:: images/6.png

#. **Core Components** 画面で、デフォルトの **Virtual Desktop Agent** に加えて **Citrix Workspace App** を選択し **Next** をクリック。

   .. figure:: images/6b.png

#. **Additional Components** 画面で、デフォルトに加えて **Citrix User Personalization Layer** を選択し **Next** をクリック。

   .. figure:: images/7.png

#. **Delivery Controller** 画面で、ドロップダウンから **Let Machine Creation Services do it automatically** を選択し、 **Next** をクリック。

   .. figure:: images/8.png

# **Features** 画面で **Next** をクリック。

   .. figure:: images/9.png

#. インストーラーが推奨するファイアーウォール設定をそのまま適用し **Next** をクリック。

#. **Install** をクリックしVDAのインストールを開始する。 （インストールプロセスは5分ほど要します。）

#. 次の画面に移行したら **Collect diagnostic information** の選択を解除し **Next** をクリック。

   .. figure:: images/10.png

#. **Finish** をクリックしVMの再起動を待ちます。

Citrix Optimizerの実行
++++++++++++++++++++++++

#. VMコンソール内でブラウザを起動し、http://10.42.194.11/workshop_staging/CitrixOptimizer.zip を入力してダウンロード。

VMコンソール内はUSキーボード配置になっているので注意。
[:] -> [Shift + ;] 、 [ _ ] -> [Shift + =]

#. **CitrixOptimizer.exe** を右クリックし **Run as Administrator** をクリック。

   .. figure:: images/12.png

#. 3.	ゴールドイメージに使用されているWindowsビルドに基づいて、推奨される(Recommendedと表示の)テンプレートをクリックします。

   .. figure:: images/13.png

#. **Select All** を選択し、クリックして、使用可能なすべての最適化を選択し、 **Analyze** をクリックします。

   .. figure:: images/14.png

#. **View Results** をクリックすると、利用可能な各最適化のステータスの詳細レポートを表示できます。

#. **Citrix Optimizer** に戻り、 **Done > Optimize** をクリックして、選択した最適化を適用します。

   .. figure:: images/15.png

#. ツールが完了したら、 **View Results** をクリックして、更新されたレポートを表示できます。 **Done** をクリックし、Window右上×ボタンでツールを閉じます。

VMware OS Optimization Toolの実行
+++++++++++++++++++++++++++++++++++

#. VMコンソール内でブラウザを開き、 http://10.42.194.11/workshop_staging/VMwareOSOptimizationTool.zip にアクセス、ダウンロードし、ダウンロードディレクトリ内に展開します。

VMコンソール内はUSキーボード配置になっているので注意。
[:] -> [Shift + ;] 、 [ _ ] -> [Shift + =]


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
