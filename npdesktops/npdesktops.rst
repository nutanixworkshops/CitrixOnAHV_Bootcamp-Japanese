.. _citrixnpdesktops:

----------------------------------
非永続デスクトップの配信
----------------------------------

永続デスクトップとは異なり、非永続デスクトップは、VMの再起動後も変更を永続化しません。非永続デスクトップの実装は、使用されるプロビジョニングテクノロジーによって異なる場合がありますが、一般に、ユーザーセッションが終了すると、VMは元の状態に戻ります。非永続デスクトップは、マスターイメージへの変更を多数のVMに一貫した方法で迅速にロールアウト（およびロールバック）できるため、管理操作を簡素化できます。非永続的なデスクトップは、従来のデスクトップまたは永続的な仮想デスクトップで時間の経過とともに発生する可能性がある負のソフトウェアおよびパフォーマンスのクリープも排除できます。

非永続デスクトップのステートレスな性質により、このアプローチはすべてのユースケースで実行できるとは限りません。カスタマイズが不要なユースケースでは、非永続デスクトップが最適です。アプリケーション設定などの永続的なエンドユーザーのカスタマイズが重要なユースケースでは、プロファイル管理ソリューションを評価して使用する必要があります。同様に、エンドユーザーデータを保持する必要がある場合、追加のネットワークベースのストレージが必要になります。組織全体のさまざまな要件やアプリケーションスタックに起因する多くのマスターイメージを維持する必要性は、大規模な非永続的なデスクトップにとっても課題となります。これらの場合、アプリケーション仮想化、アプリケーションレイヤリング、およびサーバーベースのアプリケーションテクノロジーを適用して、マスターイメージのスプロールを統合できます。

**この演習では、先の演習と同じゴールドイメージを使用して、AHV上のCitrixで非永続仮想デスクトップのプールを展開します。**

マシンカタログの作成
++++++++++++++++++++++++++++

#. **Citrix Studio**で、 **Machine Catalogsを右クリック > Create Machine Catalog** を選択する。

#. **Next** をクリック。

#. 利用可能なOSの種類を確認し、 **Single-Session OS** を選択し **Next** をクリックする。

#. **Machines that are power managed** 、 **Citrix Machine Creation Services (MCS)** を選択。 **Resources** でSecondary networkを含むリソースプールを選択する。

#. **Next** をクリック。

#. **I want users to connect to a new (random) desktop each time they log on** を選択し、**Next** をクリックする。

   .. figure:: images/1.png

#. **Default** のNutanix ストレージコンテナを選択し、 **Next** をクリックする。

#. *Initials* **Post VDA Install** スナップショットを選択し、 **Next** をクリックする。

   .. note::

     永続デスクトップのマシンカタログにて作成されたPreparationVMのスナップショットを確認してください。これらのスナップショットは、それらを使用してプロビジョニングされた仮想デスクトップがある限り存在し続けます。

#. 以下を入力して **Next** をクリックする。

   - **How many virtual machines do you want to create** - 2
   - **Total memory (MB) on each machine** - 4096
   - **Virtual CPUs** - 2
   - **Cores per vCPU** - 2

#. 以下を入力して **Next** をクリックする。

   - **Create new Active Directory accounts** を選択。
   - **Default OU** を選択。
   - **Account naming scheme** - *Initials*\ -NPD-#

   .. figure:: images/2.png

#. イニシャルを含む **Machine Catalog name** を設定し (e.g. **XYZ Windows 10 Non-Persistent 4vCPU 4GB**)  **Finish** をクリックする。

   MCSは、 *Initials*\ **-GoldImage**  VMのスナップショットからクローンを作成し、前の演習と同様の準備プロセスに従います。

#. 完了したら **Citrix Studio** で詳細を確認する。

   クローンは **Prism** に存在しますが、電源が入っていないことを確認してください。 1つのVMを選択し、Prism Elementの VMテーブルの下にある **Virtual Disks** タブで、VMに接続されているOS用vDiskとIDディスクの両方を確認します。永続的なマシンカタログと同様に、各VMには、ゴールドイメージの独自の読み取り/書き込みコピーがあるように見えます。複数のNutanixノードにまたがるマシンカタログ内のVMでは、VM読み取りのデータローカリティは、本質的にユニファイドキャッシュによって提供されます。

   このMCS実装はAHVに固有です。非永続的なマシンカタログの場合、他のハイパーバイザーは、差分ディスクと呼ばれる別のディスクへの読み取りおよび書き込みのベースゴールデンイメージにリンクします。これらのシナリオでは、Nutanixシャドウクローンを使用して、VM読み取りのデータの局所性を提供します。シャドウクローンは、マルチリーダーvDiskの分散キャッシュを自動的に提供する機能です。

   .. note:: MCSプロビジョニングの詳細については、次の記事を参照してください。:

     - `AHV向けCitrix MCS <http://blog.myvirtualvision.com/2016/01/14/citrix-mcs-for-ahv-under-the-hood/>`_
     - `o	Nutanix上のCitrix MCSおよびPVS：NutanixによるXenDesktop VMプロビジョニングの強化  <http://next.nutanix.com/t5/Nutanix-Connect-Blog/Citrix-MCS-and-PVS-on-Nutanix-Enhancing-XenDesktop-VM/ba-p/3489>`_

     Nutanixがシャドウクローンを実装する方法の詳細については、Nutanix Bibleの `シャドウクローン <https://nutanixbible.com/#anchor-book-of-acropolis-shadow-clones>`_ セクションを確認してください。

デリバリーグループの作成
+++++++++++++++++++++++++++

#. **Delivery Groupsを右クリックし > Create Delivery Group** を選択する。

#. **Next** をクリックする。

#. **Machines** ページで、 **Non-Persistent** マシンカタログを選択し、デリバリーグループのために利用可能な仮想マシンの最大数を指定する。

   .. figure:: images/3.png

#. **Users** ページで、 **Restrict use of this Delivery Group to the following users** を選択し、**Add** をクリックする。

#. **Object names** で **SSP Operators** と **devuser01** を指定して、 **OK** をクリックする。

   .. figure:: images/4.png

#. **Applications** ページで **Next** をクリックする。

#. **Desktops** ページで、 **Add** をクリックし以下を入力する:

   - **Display name** - *Initials* Pooled Win10 Desktop
   - **Description** - Non-Persistent 4vCPU/4GB RAM Win10 Virtual Desktop
   - **Allow everyone with access to this Delivery Group** を選択する。
   - **Enable desktop assignment rule** を選択する。

   .. figure:: images/5.png

   .. note::

      特定のユーザーとアプリのみにデスクトップアクセスを制限

#. **OK > Next** をクリック。

#. デリバリーグループに名前 (例 *Initials* **Win10 Non-Persistent Delivery Group**) を設定し、 **Finish** をクリックする。

#. プールの作成後、 **Prism** で1つの *Initials*\ **-NP-#** VMs の電源がONであることを確認する。

#. **Citrix Studio** で、デリバリーグループを右クリックし、 **Edit Delivery Group** を選択。

   .. figure:: images/6.png

#. サイドバーで **Power Management** を選択する。

#. ピーク時間中にパワーオンされるマシンの数について、Machinesをクリックして、1から2にドラッグします。 ピーク時間の期間はオプションで、クリックしてドラッグすることで変更できます。

   .. figure:: images/7.png

   .. note::

      For more granular control of registered, powered on VMs you can click the Edit link and provide the number or percentage of VMs you want available for every hour of the day. You can also configure the disconnected VM policy to free up disconnected VMs after a configurable time out period, returning the desktop to the pool for another user.
      パワーオン状態の登録済みVMをより細かく制御するには、Editをクリックして、1時間ごとに使用可能なVMの数または割合を指定します。また、タイムアウト期間後に切断されたVMを解放し、デスクトップを別のユーザーのプールに戻すように、切断されたVMポリシーを構成できます。

#. パワーオン状態のVMを増やした後、 **W10NP-##** VMs が **Prism** でパワーオン状態になり **Citrix Studio** に登録されたことを確認する。

   .. figure:: images/8.png

デスクトップへの接続
+++++++++++++++++++++++++

#. *Initials*\ **ToolsVM**, でブラウザを起動し、 http://ddc.ntnxlab.local/Citrix/NTNXLABWeb を入力し、Citrix StoreFront serverにアクセスする。

#. 以下の資格情報を入力し **Log On** する:

   - **Username** - NTNXLAB\\devuser01
   - **Password** - nutanix/4u

#. **Desktops** タブを選択し、２種類のデスクトップが利用できることを確認する。 **Pooled** desktop をクリックし、セッションを開始する。

   .. figure:: images/9.png

#. 仮想デスクトップへのログイン後、アプリケーションの設定を変更し、アプリケーションをインストールし、VMを再起動して、再度ログインしてみてください。operator01としてログインしてみてください、永続デスクトップと違いはありますか？

   .. note::

      ユーザーはローカル管理者グループのメンバーではないため、特定のアプリケーションをインストールできない場合があります。アプリケーションのインストール中にエラーが発生した場合は、Shiftキーを押しながらインストーラーを右クリックし、[ 別のユーザーとして実行 ]を選択します。NTNXLAB \ Administrator資格情報を使用して、インストールを完了します。

お持ち帰り
+++++++++

- MCSでは、単一のゴールドイメージを永続的なマシンカタログと非永続的なマシンカタログの両方に使用できます。

- 非永続的な仮想デスクトップは、ユーザーがログインするたびに「新しい」VMを取得するため、一貫したエクスペリエンスを提供します。このアプローチは、従来のソフトウェアパッチ適用よりも大幅な操作の節約を提供できますが、非永続デスクトップ上で必要なカスタマイズを提供するために他のツールが必要になる可能性があります。売店や教育期間などのユースケースは、非永続デスクトップに最適です。

- シンカタログ内のすべてのVMは単一の共有ゴールドイメージに基づいていますが、データの局所性（読み取りの遅延の削減とネットワークの輻輳の削減）の恩恵を受け続けています。非AHVハイパーバイザーの場合でも、シャドウクローンを通じて同じ利点が実現されます。
