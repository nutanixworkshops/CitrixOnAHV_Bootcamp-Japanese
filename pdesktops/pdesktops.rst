.. _citrixpdesktops:

------------------------------
永続デスクトップの配信
------------------------------

従来の物理デスクトップPCと同様に、永続的な仮想デスクトップは、再起動後もVMへの変更を保持します。永続デスクトップは、リモートプロファイルに依存せずに独自のアプリケーションをインストールしたり、OSやアプリケーションの設定をカスタマイズしたりできるなど、ユーザーに最高の柔軟性を提供します。ただし、VMのプロビジョニングが完了すると、SCCM、WSUS、または他のサードパーティのパッチ管理ツールなどの従来の手段によるパッチ適用/更新が必要になり、VDIを実装する運用上の利点が制限されます。

Citrixは、マスターイメージの完全なバイトコピークローンを作成するのではなく、Machine Creation Services（MCS）を使用して、共有マスターイメージから永続デスクトップを効率的にプロビジョニングできます。MCSは、Desktop Delivery Controllerの役割の一部としてインストールされ、Citrix Studioで管理されるVM作成/オーケストレーションフレームワークです。

**このラボでは、AHV上のCitrixを使用して永続仮想デスクトップのプールを展開およびテストします。**

AHVリソースの設定
+++++++++++++++++++++++++

演習環境には、 **XenDesktop Delivery Controller（DDC）** が、割り当てられたクラスターの共有リソースとしてインストールされています。 DDCはXenDesktopの接続ブローカーとして機能し、Windowsサーバーに展開されます。またこのDDC VMには、Citrix Webフロントエンド（StoreFront）とCitrix Licensingのサービスが含まれています。どちらも実際の運用環境で実行されるコンポーネントです。

XenDesktopサイトには、スケールアウトして大規模化する環境に対応するために、複数のDelivery ControllerとStoreFrontを構築することができます。

#. *Initials*\ **-WinToolsVM** VM へRDPで接続する:

   - **User Name** - NTNXLAB\\Administrator
   - **Password** - nutanix/4u

#. スタートメニューから **Citrix Studio** を選択する。

   Citrix Studioは、Delivery Controller、StoreFront、LicensingなどのXenDesktopサイトの管理に使用されるMMCスナップインです。デフォルトでDDCにインストールされますが、同一ネットワーク上の任意のマシンのクライアントとして実行することもできます。

#. 環境内に存在するXenDesktop Controllerの役割を持つ **ddc.ntnxlab.local** を指定し **Connect** をクリックする。

   .. figure:: images/1.png

#. **Citrix Studio > Configuration > Hosting** を選択し、 **NutanixAcropolis** をクリックします。 **Details** ペインに接続設定がされたPrism Elementの仮想IPが表示されます。

   .. figure:: images/2.png

   .. note::

      XenDesktopはデフォルトでVMware vSphere、Microsoft Hyper-V、Citrix XenServer、Microsoft Azure、AWSなどの多くのプラットフォームへの仮想マシンのプロビジョニングをサポートしています。Citrix Provisioning SDKは、Delivery Controllerのプロビジョニングおよび電源管理機能と追加のプラットフォームを統合する機能を提供し、Nutanix AHV上でネイティブのCitrix管理エクスペリエンスを作成します。

      この統合を可能にするために、 **Nutanix AHV Plugin for Citrix** が必要であり、本環境ではDDCにプリインストールされています。実稼働環境では、このプラグインを各DDCにインストールする必要があります。プラグインは、 `Nutanix Portal <https://portal.nutanix.com/#/page/static/supportTools>`_ からダウンロードできます。

      .. figure:: images/5.png

#. **NutanixResources** を選択し、 **Primary** ネットワークだけが利用可能になっていることを確認する。 左メニューから **Hosting** を選択し、 **Actions** メニューから **Add Connection and Resources** をクリックする。

#. **Use an existing Connection: NutanixAcropolis** を選択し、 **Next** をクリックする。

   .. figure:: images/3.png

#. 以下を入力し **Next > Finish** をクリックする。:

   - **Name for these resources** - *Initials*\ -Resources
   - Select **Secondary Network**

   .. figure:: images/4.png

マシンカタログの作成
++++++++++++++++++++++++++++

マシンカタログは、物理マシンまたは仮想マシンのコレクションです。 MCSまたはPVSを使用してゴールドイメージからマシンカタログをプロビジョニングする場合、そのイメージからプロビジョニングされたすべてのマシンは、同じVM構成（vCPU、メモリ、ネットワーク）を共有し、同じドメインに属します。単一のゴールドイメージを複数のマシンカタログに使用して、異なるサイズのVM、複数のドメインにまたがるVMなどを提供できます。

#. **Citrix Studio** で **Machine Catalogs** を右クリックし **Create Machine Catalog** を選択。

   .. figure:: images/6.png

#. **Next** をクリックする。

#. 利用可能なOSを確認し、 **Single-Session OS** を選択し **Next** をクリックする。

   .. figure:: images/7.png

#. **Machines that are power managed** を選択し **Citrix Machine Creation Services (MCS)** を選択する。また **Resources** で先ほど作成したリソースプールを選択する。

   .. figure:: images/8.png

#. **Next** をクリックする。

#. **Static** を選択し、また **Yes, create a dedicated virtual machine** を選択し **Next** をクリックする。

   .. figure:: images/9.png

#. **Default** のNutanix ストレージコンテナを選択し、 **Next** をクリックする。

   .. note::

      仮想デスクトップを実行しているすべてのストレージコンテナーには、圧縮（インラインまたはポストプロセス）をお勧めします。重複排除は、フルバイトコピーのクローンVMを実行しているストレージコンテナーに対してのみ推奨されます。

#. 先ほど取得したゴールドイメージのスナップショット *Initials* **Post VDA Install** を選択し **Next** をクリックする。

   .. figure:: images/10.png

#. 以下を入力し **Next** をクリックする。:

   - **How many virtual machines do you want to create** - 2
   - **Total memory (MB) on each machine** - 4096
   - **Virtual CPUs** - 2
   - **Cores per vCPU** - 2

#. 以下を入力し **Next** をクリックする。:

   - **Create new Active Directory accounts** を選択
   - **Default OU** OUを選択
   - **Account naming scheme** - *Initials*\ -PD-#

   マシンカタログの作成の一環として、Delivery ControllerはADにすべてのマシンアカウントを作成します。クローンされたVM自体は従来のSysprepおよびドメイン参加を経由しないため、この処理が必要です。代わりに、（VDAの一部としてインストールされる）Citrix Machine Identity ServiceがVMの「一意性」を管理し、デスクトップリソースの大規模なプールをプロビジョニングするより迅速な手段を提供します。

   .. figure:: images/11.png

#. イニシャルを含む **Machine Catalog name** を設定し (例 **XYZ Windows 10 Persistent 4vCPU 4GB**)  **Finish** をクリックする。

   MCSは、 *Initials*\ **-GoldImage** のスナップショットからクローンを作成します。MCS利用時、Delivery Controllerは構成済みの各データストアにゴールドイメージをコピーします。従来のSANシナリオ（またはローカルストレージでMCSを使用）では、マシンカタログが複数のボリュームに分散されるため時間がかかる可能性があります。一方Nutanixクラスターでは、すべてのデスクトップにサービスを提供する単一のデータストア（ストレージコンテナー）を構成するため、マシンカタログのプロビジョニング時間を短縮します。

   .. figure:: images/12.png

   自動的に削除される前に、 **Prism** で起動中のPreparation クローンを確認します。このVMには、VMがマシンカタログで使用できるようにするための複数の手順を実行する個別のディスクが接続されています。

   Preparationの段階では、DHCPを有効にし、Windowsライセンスの「再準備」を実行して、それが一意のVMとしてMicrosoft KMSサーバーに報告されるようにし、同様にOfficeライセンスの「再準備」を実行します。 Citrix Studioは、Preparation が完了してシャットダウンすると、この状態のVMのスナップショットを自動的に作成します。

   .. figure:: images/13.png

   MCSは、マシンカタログ用のVMを作成します。これには、VMとクローンベースvDiskの作成、およびIDディスクと呼ばれる小さな（最大16MB）vDiskの作成が含まれます。 IDディスクには、ホスト名とActive Directoryマシンアカウントパスワードを提供する各VMに固有の情報が含まれています。この情報はCitrix Machine Identity Serviceによって自動的に取り込まれ、VMが一意として表示され、ドメインに参加できるようになります。

   .. figure:: images/14.png

    **Prism** にクローンが存在しており、また電源が入っていないことを確認します。 VMの1つを選択し、 **Prism Element** のVMテーブルの下にある **Virtual Disks** タブで、VMに接続されているOS用 vDiskとIDディスクの両方を確認します。各VMは、ゴールドイメージの独自の読み取り/書き込みコピーを持っているように見えます。複数のNutanixノードにまたがるマシンカタログ内のVMでは、VM読み取り時のデータローカリティは本質的にユニファイドキャッシュによって提供されます。

   .. note:: Nutanixユニファイドキャッシュの動作の詳細については、Nutanix Bibleの ` I/Oパスとキャッシュ <http://nutanixbible.com/#anchor-i/o-path-and-cache-65> `_ セクションを参照してください。

   .. figure:: images/pdesktops8.png

#. 完了したら、 **Citrix Studio** でマシンカタログの詳細を表示します。

   .. figure:: images/16.png

デリバリーグループの作成
+++++++++++++++++++++++++++

デリバリーグループは、マシンカタログに存在する1つ以上のマシンのコレクションです。デリバリーグループの目的は、マシンにアクセスできるユーザーまたはグループを指定することです。永続的なデスクトップの場合、マシンとユーザーアカウントの間に永続的な関係が作成されます。この割り当ては、デリバリーグループの作成時に手動で行うか、ユーザーの最初のログオン時に自動的に割り当てることができます。

#. **Citrix Studio** で **Delivery Groupsを右クリック > Create Delivery Group** を選択。

   .. figure:: images/17.png

#. **Next** をクリック。

#. 作成した **Persistent** マシンカタログを選択し、デリバリーグループで使用可能なVMの最大数を指定。

   .. figure:: images/18.png

#. **Delivery Type** で **Desktops** を選択し、 **Next** をクリック。

   .. note::

      Citrixは、共有サーバーOSで実行されるアプリケーションの配信でよく知られていますが、デスクトップOSを使用して、完全なデスクトップエクスペリエンスを提供せずにシームレスなアプリケーションを配信することもできます。通常このアプローチは、サーバーOSを介したアプリケーションの配信を妨げるライセンスの問題がある場合や、一度に1人のユーザーのみがそのリソースにアクセスできるVMで実行することによりアプリケーションのパフォーマンスを分離するために使用されます。

#. **Restrict use of this Delivery Group to the following users** を選択し、 **Add** を選択。

#. **Object names** で **SSP Developers** を指定し **OK** をクリック。

   .. figure:: images/19.png

#. **Next** をクリック。

#. **Add** をクリックし、以下を入力する。:

   - **Display name** - *Initials* Personal Win10 Desktop
   - **Description** - Persistent 4vCPU/4GB RAM Windows 10 Virtual Desktop
   - **Allow everyone with access to this Delivery Group** を選択
   - **Maximum desktops per user** - 1
   - **Enable desktop assignment rule** を選択

#. **OK > Next** をクリック。

   デリバリーグループに名前を設定し (例 *Initials* **Win10 Persistent Delivery Group**)  **Finish** をクリック。

#. プールの作成後、 **Prism** で、 *Initials*\ **-PD-#** の一つがパワーオン状態であることを確認する。

#. **Citrix Studio** で 作成したデリバリーグループを右クリックし **View Machines** をクリック。 （もしくは、デリバリーグループをダブルクリック。）

#. デスクトップがDelivery Controllerに **Registered** として電源オン後すぐに表示され、ユーザー接続の準備ができていることを確認します。

..   .. figure:: images/20.png

デスクトップへの接続
+++++++++++++++++++++++++

#. *Initials*\ **ToolsVM** にてブラウザを開き http://ddc.ntnxlab.local/Citrix/NTNXLABWeb を入力しCitrix StoreFront serverにアクセスする 。

#. **Detect Receiver** をクリックし、 ライセンスに承諾し **Download** をクリックし、**Citrix Workspace App** をダウンロードする。

   .. figure:: images/21.png

   .. note::

      Citrixデスクトップおよびアプリケーションにアクセスするためのクライアントとして使用されるCitrix Workspaceアプリケーションは、以前はCitrix Receiverと呼ばれていました。

#. **CitrixWorkspaceApp.exe** インストーラーを起動し、完了します。 ※Single Sign-On を有効に **しない** 。

#. インストール完了後、ブラウザに戻り **Continue** をクリック。

   .. figure:: images/22.png

#. Chromeでプロンプトが表示されたら **Open Citrix Workspace Launcher** をクリック。

   .. figure:: images/23.png

#. 以下の認証情報を入力し **Log On** をクリック:

   - **Username** - NTNXLAB\\devuser01
   - **Password** - nutanix/4u

#. **Desktops** タブを選択し、 **Personal Win10 Desktop** をクリックし、セッションを起動する。

   .. figure:: images/24.png

   .. note::

     ブラウザーによっては、Receiverが自動的に開かない場合、ダウンロードした.icaファイルをクリックする必要がある場合があります。また、ブラウザに.icaファイルを常に開くように指示できる場合もあります。

#. 仮想デスクトップのログインが完了したら、アプリケーション設定の変更、アプリケーションのインストール、VMの再起動、および再度ログインなどを実験する。

   .. note::

      ユーザーはローカル管理者グループのメンバーではないため、特定のアプリケーションをインストールできない場合があります。アプリケーションのインストール中にエラーが発生した場合は、Shiftキーを押しながらインストーラーを右クリックし、[ 別のユーザーとして実行 ]を選択します。NTNXLAB \ Administrator資格情報を使用して、インストールを完了します。

      .. figure:: images/26.png

#. **Citrix Studio** で、 VMの詳細情報への変更を確認する。 ユーザーがログインすると、デスクトップが静的に割り当てられ、別のデスクトップの電源がオンになり、Delivery Controllerに登録されて、次のユーザーを待ちます。

   .. figure:: images/25.png

お持ち帰り
+++++++++

- Citrixは、HTML5を介して高忠実度のデスクトップエクスペリエンスを提供できます。同様に、HTML 5 Nutanix Prismインターフェイスは、どこからでもインフラストラクチャを管理および監視するための単一のUIを提供します。

- 単一のストレージコンテナから大規模な環境をサポートする機能により、構成が簡素化され展開速度が向上します。

- マシンカタログ内のすべてのVMは単一の共有ゴールドイメージに基づいていますが、データローカリティ（読み取りの遅延の削減とネットワークの輻輳の削減）の恩恵を受け続けています。非AHVハイパーバイザーの場合でも、シャドウクローンを通じて同じ利点が実現されます。

- インテリジェントなクローン作成により、永続的な仮想デスクトップを展開するための大きなストレージオーバーヘッドが回避されます。同じクラスター内で永続デスクトップと非永続デスクトップを混在させる場合、ベストプラクティスは、永続デスクトップ用に重複排除を有効にしたストレージコンテナーと、非永続デスクトップ用に重複排除を無効にした別のストレージコンテナーを活用することです。ワークロードを適切なストレージ効率技術と組み合わせる柔軟性があると、密度が向上し無駄が減ります。

- Citrix MCSでは、単一のコンソールでエンドツーエンドのプロビジョニングと資格管理が可能です。

- 永続的な仮想デスクトップは、ユーザーがデスクトップエクスペリエンスを完全に制御できる従来のデスクトップのようなエクスペリエンスを提供します。このアプローチは少数のユーザーのサブセットに必要な場合がありますが、通常、レガシーソフトウェアのパッチツールに依存し続けるため、大規模には望ましくありません。
