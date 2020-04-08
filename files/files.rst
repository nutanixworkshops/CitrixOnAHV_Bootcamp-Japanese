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

#. **Security** タブを開き、 **Advanced** をクリックする。

#. デフォルトの **Users (BootcampFS\\Users)** をエントリーを選択し **Remove** をクリックする。

#. **Add** をクリック。

#. **Select a principal** をクリックし、 **Object Name** で **Everyone** を指定して **OK** をクリックする。

#. 以下を入力し **OK** をクリックする。:

   - **Type** - Allow
   - **Applies to** - This folder only
   - **Read & execute** を選択する。
   - **List folder contents** を選択する。
   - **Read** を選択する。
   - **Write** を選択する。

   .. figure:: images/5.png

#. **OK > OK > OK** と進む。

   .. figure:: images/6.png

Citrixユーザープロファイル管理(UPM)の構成
++++++++++++++++++++++++++++++++++++++++++

UPM（User Profile Management）は、仮想デスクトップまたはXenAppサーバー内のVirtual Delivery Agentの一部としてインストールされたシステムサービスとして実行されます。Microsoft Roaming Profilesに似ていますが、プロファイルをオンデマンドでストリーミングすることによるログオンの高速化、プロファイルサイズを制限するための管理コントロール、詳細なログなどの重要な利点があります。

この演習では、Microsoftグループポリシーと同様に、Citrixポリシーエンジンを介してUPMを有効にします。

#. **Citrix Studio > Policies** と進み、 **Policiesを右クリック > Create Policy** をクリックする。

   .. figure:: images/7.png

#. **All Settings** のドロップダウンから、 **Profile Management > Basic Settings** と進む。（オプション： **All Versions** のドロップダウンから **1912 Single-Session OS** でサポートされるポリシーのみをフィルターすることができます。）

   .. figure:: images/8.png

#. **Enable Profile management** を検索し、 **Select** を選択。 **Enabled** を選択し、 **OK** をクリックする。

   .. figure:: images/9.png

#. **Path to user store** を検索し、 **Select** をクリック。 **Enabled** を選択し、パスに ``\\BootcampFS\Initials-CitrixProfiles\%USERNAME%\!CTX_OSNAME!!CTX_OSBITNESS!`` を指定する。 **OK** をクリックする。

   .. figure:: images/10.png

   .. note::

     指定されたパスは、各ユーザーの共有内に一意の最上位ディレクトリを作成するだけでなく、例えばWindows 10ユーザープロファイルをWindows 2012セッションに適用しようとするなどの非互換操作を回避するために、プロファイル用のプラットフォーム固有のサブディレクトリも作成します。

#. **Next** をクリックする。

#. **Delivery Group** の右側にある **Assign** をクリックする。

#. **Delivery Group** のドロップダウンメニューから、非永続デリバリーグループを選択し、 **OK** をクリックする。

   .. figure:: images/11.png

   .. note::

     Studioには、ポリシーを適用するさまざまな方法が用意されています。より多様な環境では、OUまたはタグに基づいてUPM設定を構成することが理にかなっています。

#. **Next** をクリックする。

#. **Policy name** を設定し(例 *Initials*\ **-UPM**)  **Enable policy** を選択する。 設定内容を確認し、 **Finish** をクリックする。

   .. figure:: images/12.png

プロファイルとフォルダーリダイレクトのテスト
+++++++++++++++++++++++++++++++++++++++

#. *Initials*\ **ToolsVM** にてブラウザを起動し、 http://ddc.ntnxlab.local/Citrix/NTNXLABWeb にアクセスし **NTNXLAB\\operator02** でログインして **Pooled Windows 10 Desktop** に接続する。

#. 仮想デスクトップにて、ファイルやフォルダーを作成するなどの小さな変更を加える。※接続しているデスクトップのホスト名を控えておく。

   .. figure:: images/afsprofiles15.png

#. **PowerShell** を起動し、以下のコマンドを実行しブロックタイプのファイルを作成する。:

   .. code-block:: PowerShell

      New-Item \\BootcampFS\INITIALS-CitrixProfiles\operator02\Win10RS6x64\UPM_Profile\Documents\test.mp3

   新しいファイル作成が拒否されることを確認する。

#. **Pooled** デスクトップからサインアウトする。 ※Citrix Workspace セッションを閉じないでください。

#. 再度Citrix StoreFrontへ **NTNXLAB\\operator02** でログインし、**Pooled Windows 10 Desktop** に接続する。 ログインするたびに基盤となるデスクトップが新たにプロビジョニングされるにもかかわらず、ファイルと設定はセッションを超えて保持されることを確認する。

#. エクスプローラーで ``\\BootcampFS\Initials-CitrixProfiles\operator02`` に接続し、ユーザープロファイルに関連づけられたプロファイルを見付ける。

#. 仮想デスクトップからサインアウトする。 **Citrix Workspace Appを閉じないでください**

#. Citrix StoreFront に **NTNXLAB\\operator01** でログインし、 **Pooled Windows 10 Desktop** へ接続する。エクスプローラーで ``\\BootcampFS\Initials-CitrixProfiles\`` を開き、 **operator02** の profileディレクトリが表示されないか、アクセスできないことを確認する。　※ **Prism > File Server > Share/Export > home > Update** で、 **Access Based Enumeration (ABE)** を無効化して再度確認してみてください。

#. (オプション演習) 非永続デスクトップにて **Documents** フォルダにテキストファイルを作成する。約1時間後、仮想デスクトップに戻り、作成したテキストを編集して保存する。対象ファイルを右クリックし、 **Restore previous versions（以前のバージョン）** を選択する。 使用可能な以前のバージョンを選択し、 **Open** をクリックしてファイルにアクセスする。

.. figure:: images/afsprofiles16.png

(オプション演習) Citrix User Personalization Layers（UPL）でのFilesの利用
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Citrix Virtual Apps and Desktops（VAD）のCitrix UPL機能は、セッション間でデータとローカルユーザーインストールアプリケーション（UIA）を保持することにより、非永続的なWindows 10デスクトップの機能を拡張します。Citrix UPLは、App Layering User Layerと同じテクノロジーですが、Citrix Virtual Delivery Agent（VDA）に統合され、Citrixポリシーエンジンを使用します。Citrix UPLは、ユーザーレイヤー（UL）のすべての機能を備えており、アプリレイヤー化プロセス全体を実行したり、Enterprise Layering Manager（ELM）仮想アプライアンスを展開したりする必要はありません。

.. note::

   ユーザーが仮想デスクトップにローカルにインストールするすべてのアプリケーションは、次の項目を除き、Citrix UPLでサポートされます。:

   - Microsoft OfficeやVisual Studioなどのエンタープライズアプリケーション
   - VPNクライアントなど、ネットワークスタックまたはハードウェアを変更するアプリケーション
   - ウイルス対策プログラムなどのブートレベルのドライバーを備えたアプリケーション
   - プリンタードライバーなど、ドライバーストアを使用するドライバーを持つアプリケーション

   上記のアプリケーションをユーザーにUPLの一部として仮想デスクトップにローカルにインストールさせる代わりに、これらのアプリケーションをマスターイメージにインストールします。

   ローカルユーザーまたはグループを追加または編集しようとするアプリケーションは、変更を保持しません。代わりに、必要なローカルユーザーまたはグループをマスターイメージに追加します。

   完全な要件と推奨事項については、 `Citrixの製品ドキュメント Citrix Virtual Apps and Desktops User Personalization Layer <https://docs.citrix.com/en-us/citrix-virtual-apps-desktops/install-configure/user-personalization-layer.html>`_ を確認してください。

#. **Prism Element > File Server > Share/Export** と進み、 **+ Share/Export** をクリック。

#. **Basic** 項目にて、以下を入力し **Next** をクリックする。:

   - **Name** - *Initials*\ **-CitrixUPL**
   - **Description** - Citrix UPL storage
   - **File Server** - BootcampFS
   - **Select Protocol** - SMB

#. **Next > Create** をクリックする。

#. *Initials*\ **-WinTools** VMにて、エクスプローラーを開き ``\\BootcampFS.ntnxlab.local\`` にアクセスする。

#. *Initials*\ **-CitrixUPL** を開き、 **Users** という名前のディレクトリを作成する。

   .. figure:: images/15.png

   .. note::

      フォルダー名はCitrix UPLでハードコーディングされており、 **Users** という名前にする必要があります。

#. **Citrix Studio > Policies** に戻り、 **UPM** を右クリックし、 **Disable** をクリックする。

   UPLポリシーを同じデスクトップグループに適用します。

#. **Create Policy** をクリックする。

#.  **Search** 欄で、 **User Layer** を指定する。

   .. figure:: images/16.png

#. **User Layer Repository Path** を選択し、 *Initials*\ **-CitrixUPL** を指定し **OK** をクリックする。 ※ **Users** フォルダーをパスに含めないように注意してください。これは自動的に追加されます。

   .. figure:: images/17.png

#. **User Layer Size in GB** を選択し、 **20** GBを指定し、 **OK** をクリックする。

   .. note:: デフォルト値の0では、10GB UPLディスクが構成されます。

#. **Next** をクリックする。

#. **Delivery Group** 右側の **Assign** をクリックする。

#. **Delivery Group** のドロップダウンメニューから、非永続デリバリーグループを選択し、 **OK** をクリックする。

   .. figure:: images/11.png

   .. note::

      Citrix UPLは、Pooled-RandomおよびPooled-Static Machine Catalogと連携します。Citrix UPLは、Citrix Personal vDisk（現在廃止予定）またはローカルディスクへの変更を保存する専用の永続的なマシンを備えたPooled-Staticマシンカタログをサポートしていません。

#. **Next** をクリックする。

#. **Policy name** を設定し(例 *Initials*\ **-UPL**)  **Enable policy** を選択する。設定を確認し、 **Finish** をクリックする。

#. *Initials*\ **ToolsVM** にてブラウザを開き、 http://ddc.ntnxlab.local/Citrix/NTNXLABWeb に接続し、 **NTNXLAB\\operator03** でログインし **Pooled Windows 10 Desktop** に接続する。

#. エクスプローラーにて ``\\BootcampFS.ntnxlab.local\<Initials>-CitrixUPL\Users`` へ接続する。パーソナルデスクトップレイヤーのVHDを含むユーザーディレクトリがあることを確認する。

   .. figure:: images/18.png

#. 仮想デスクトップ上で **Mozilla Firefox** をインストールし、デフォルトブラウザとして設定する。

#. 仮想デスクトップを再起動する。

#. 2分ほど経過した後、Citrix StoreFront にて別の **Pooled Windows 10 Desktop** を起動する。FireFoxがインストールされており、デフォルトブラウザとして設定されていることを確認する。同様に、FireFoxを起動し、初期セットアップが不要であることを確認する。

   .. figure:: images/19.png

#. 仮想デスクトップから切断する。

お持ち帰り
+++++++++

- •	Nutanix Filesは、ユーザープロファイル、データ、およびCitrix User Personalization Layer VHDファイルの保存に適したネイティブファイルサービスを提供します。

- •	Citrix User Personalization Layerは、非永続的なプロビジョニングおよびMachine Creation Servicesイメージ用のApp Layering User Layerの簡易バージョンです。

- Nutanix Filesは、Citrix仮想デスクトップと同じNutanixクラスターに展開できるため、ストレージ容量の利用率が向上し、追加のストレージサイロがなくなります。

- •	混合ワークロード（仮想デスクトップやファイルサービスなど）のサポートは、次のような単一クラスター内で異なるノード構成を混合するNutanixの機能によってさらに強化されます。:

  - ストレージヘビーなノードと、コンピュートヘビーなノードの混在
  - 追加の仮想化ライセンスコストを発生させることなく、ストレージ専用ノードを追加することでストレージ容量を増やす
  - 異なる世代のハードウェアの混合（例：NX-3460-G6 + NX-6235-G5）
  - オールフラッシュノードとハイブリッドノードの混在
  - NVIDIA GPUノードと非GPUノードの混合
