.. _citrixgoldimage:

------------------------------------
ゴールドイメージの構築と最適化
------------------------------------

Windows Client OSは通常物理デバイス上（ラップトップPCなど）で動作することを前提としており、物理ハードウェアデバイスに直接接続され、
また同ホスト上の負荷が高い仮想マシンに影響を受けるなどが発生しないことを想定しております。
そのため、そのOSを仮想マシンとしてインストールすると想定外の動作をする可能性があり、仮想環境上で動作させるための最適化設定を行う事で
パフォーマンスの向上や集約率の向上を図ることができます。

この演習では、Windows 10 VMにCitrix Virtual Desktop Agentをインストールし、Citrix OptimizerとVMware OS Optimization Toolを使用してWindows OSを最適化します。
Windows OSを仮想デスクトップとして動作させる際の最適化の1つの方法として、今回2つのツールを使用します。
実案件の場合は要件やCitrixのベストプラクティスを参照頂き、検証を実施した上で最適化設定を行ってください。ツールを使った最適化は必須ではありません。

ゴールドイメージ用VMの構築
++++++++++++++

#. Prism Centralで :fa:`bars` > **仮想インフラ（Virtual Infrastructure）** > **仮想マシン（VMs）** を選択。

   .. figure:: images/1.png

#. **仮想マシンを作成（Create VM）** をクリック。

#. 以下を入力する:

   - **Name** - *Initials*\ -GoldImage
   - **Description** -
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

Windows 10 ゴールドイメージの構築を開始する前に、Windows Updateが進行中でないことを確認することが重要です。Windows Updateによりクローン作成で問題が発生する可能性があります。
本演習ではWindows Updateを一次停止しゴールドイメージを作成します。実案件の場合は適用完了後ゴールドイメージを作成ください。

#. 作成したVMのチェックボックスをクリックし **Actions（アクション） > Launch Console（コンソールの起動）**

   - **User Name** - Nutanix
   - **Password** - nutanix/4u

#. **System Settings > Windows Update** を開き **Pause Updates for 7 Days** をクリックする。

   .. figure:: images/24.png

#. VMを再起動する。

VDAのインストール
++++++++++++++++++

Virtual Delivery Agent（VDA）は、ユーザー各物理マシンまたは仮想マシンに接続時に必要なドライバーとサービスのソフトウェアです。
VDAにより、仮想デスクトップ用のマシンをDelivery Controllerに登録します。また、Delivery Controllerがこれらのマシンをユーザーに割り当てることができます。
VDAは、マシンとユーザーデバイス間のHDX接続、Citrixリモートプロトコルの確立、ライセンスの確認、Citrixポリシーの適用も担当します。

#. VMのコンソールで再起動完了を確認する。

#. **Computer Name** (e.g. *Initials*\ -GoldImage) を変更し以下の資格情報で **NTNXLAB.local** に参加する:

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

#. **Update > Save** をクリック。

#. ゴールドイメージVMのコンソールにて **D:\\AutoSelect.exe** を開きCitrix installerを起動。

   .. figure:: images/3.png

#. **Virtual Apps and Desktops > Start** を選択

   .. figure:: images/4.png

#. **Prepare Machines and Images** を選択し、Virtual Delivery Agentのインストールを開始する。

   .. figure:: images/5.png

#. **Create a MCS master image** を選択し **Next** をクリック。

   .. figure:: images/6.png

#. **Core Components** 画面で、デフォルトの **Virtual Desktop Agent** に加えて **Citrix Workspace App** を選択し **Next** をクリック。

   .. figure:: images/6b.png

#. **Additional Components** 画面で、デフォルトに加えて **Citrix User Personalization Layer** を選択し **Next** をクリック。

   .. figure:: images/7.png

#. **Delivery Controller** 画面で、ドロップダウンから **Let Machine Creation Services do it automatically** を選択し、 **Next** をクリック。

   .. figure:: images/8.png

#. **Features** 画面で **Next** をクリック。

   .. figure:: images/9.png

#. インストーラーが推奨するファイアーウォール設定をそのまま適用し **Next** をクリック。

#. **Install** をクリックしVDAのインストールを開始する。 （インストールプロセスは5分ほど要します。）

#. 次の画面に移行したら **Collect diagnostic information** の選択を解除し **Next** をクリック。

   .. figure:: images/10.png

#. **Finish** をクリックしVMの再起動を待ちます。

Citrix Optimizer の実行
++++++++++++++++++++++++

#. VMコンソール内でブラウザを起動し、http://10.42.194.11/workshop_staging/ を入力してダウンロード。
  VMコンソール内はUSキーボード配置になっているので注意が必要です。
  演習環境に接続しているPCが日本語キーボードの場合は [:] -> [Shift + ;] 、 [ _ ] -> [Shift + =] で入力してください。

#. **CitrixOptimizer.zip** を展開し **CitrixOptimizer.exe** を右クリックし **Run as Administrator** をクリック。

   .. figure:: images/12.png

#. ゴールドイメージに使用されているWindowsビルドに基づいて、推奨される(Recommendedと表示の)テンプレートをクリック。

   .. figure:: images/13.png

#. **Select All** を選択し、クリックして、使用可能なすべての最適化を選択し、 **Analyze** をクリック。

   .. figure:: images/14.png

#. **View Results** をクリックすると、利用可能な各最適化のステータスの詳細レポートを表示されます。

#. **Citrix Optimizer** に戻り、 **Done > Optimize** をクリックして、選択した最適化を適用します。

   .. figure:: images/15.png

#. ツールが完了したら、 **View Results** をクリックして、更新されたレポートを表示できます。 **Done** をクリックし、Window右上×ボタンでツールを閉じる。



VMware OS Optimization Toolの実行 (オプション)
+++++++++++++++++++++++++++++++++++

※Citrix環境でありますが、OSレイヤーに対する最適化を行うツールであるためVMware OS Optimization Toolを使った最適化を図ることができます。
　実案件の場合は要件やCitrixのベストプラクティスを参照頂き、検証を実施した上で最適化設定を行ってください。
　この手順はオプションです。


#. VMコンソール内でブラウザを開き、 http://10.42.194.11/workshop_staging/VMwareOSOptimizationTool.zip にアクセス、ダウンロードし、ダウンロードディレクトリ内に展開する。
     VMコンソール内はUSキーボード配置になっているので注意。
     [:] -> [Shift + ;] 、 [ _ ] -> [Shift + =]


#. **VMwareOSOptimizationTool.zip** を展開し **VMwareOSOptimizationTool.exe** を右クリックし、 **Run as Administrator** を選択する。

#. **Select All** のチェックボックスをクリック。 **Cleanup Jobs** の項目までスクロールし、該当の4項目のチェックを外し、 **Analyze** をクリックする。

   .. figure:: images/16.png

   .. note::

      クリーンアップジョブは適用に時間がかかる為、今回の演習からは除外します。

#. **Analysis Summary** ペインで適用する最適化項目の内訳が確認できます。

   .. figure:: images/17.png

#. **Optimize** をクリックし最適化を適用する。

   .. figure:: images/18.png

#. 結果を確認して、ゴールドイメージVMを再起動する。


ゴールドイメージの完成
+++++++++++++++++++++++++

Citrix Virtual Desktop(XenDesktop)は、ゴールドイメージのスナップショットを利用してデスクトップのプールをプロビジョニングします。
従来のスナップショットはチェーン構造であり、長いスナップショットチェーンを走査するとパフォーマンスが低下する可能性がありましたが、
NutanixのスナップショットはRedirect-on-Writeアルゴリズムを採用しており、このような欠点はありません。
この違いにより、ゴールドイメージスナップショットを使用して、単一のVMから多くのゴールドイメージバージョンを維持する柔軟性が得られます。


※詳細は http://nutanixbible.jp/#anchor-スナップショットとクローン-80 を参照

#. ゴールドイメージVMの再起動完了後、仮想マシン内からシャットダウンを実行。

#. **Prism Element** からゴールドイメージVMのスナップショットを取得する。 (名前は *Initials Post optimization and VDA install*)

   .. figure:: images/20.png

   .. note::

      このスナップショットは、Citrix AHVプラグインによって認識されるために、Prism Elementから取得する必要があります。


お持ち帰り
+++++++++

この演習で学んだ重要なこと

- ゴールドイメージは仮想デスクトップを展開するための大本のイメージとなります、そのためこのイメージから展開する全ての仮想デスクトップ対して適用することができます。
　仮想デスクトップに適切な最適化を実施することで、パフォーマンスの向上や集約率の向上を図ることができます

- 最適化ツールは、ビルド毎に最適化設定がまとめられており簡単に最適化設定を行う事ができます。案件によっては必要な設定まで変更される可能性がありますので十分検証頂き使用してください。

- Citrix Virtual Desktop(XenDesktop)のMCSを使用すると、イメージを登録するDelivery Controllerを手動で指定する必要がないため、ゴールドイメージの管理を簡素化できます。
　これにより、単一のゴールドイメージが外部の依存関係なしに複数の環境をサポートする柔軟性が高まります。

- 従来の仮想化基盤ではMCSを使った展開を行うとベースイメージにI/O負荷が集中するため大規模展開が難しいとされてきましたが、Nutanix HCIのストレージ最適化の機能によってI/O負荷の集中を回避することが出来るため、MCSを使った大規模展開の事例が増えてきています。
