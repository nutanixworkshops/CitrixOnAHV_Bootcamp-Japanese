.. _citrixmanage:

------------------------------
Citrix Desktopsの管理
------------------------------

Nutanix、AHV、XenDesktopの組み合わせにより、新しいインフラストラクチャと仮想デスクトップの展開がスムーズになりますが、その環境が変化し、拡大するとどうなりますか？

**この演習では、XenDesktop環境で "Day 2" の操作を実行します。これには、より多くのデスクトップVMで既存のマシンカタログを拡張し、更新されたゴールドイメージを非永続マシンカタログに展開します。**

デスクトップの追加
++++++++++++++++++++

Nutanixを使用すると、Prismのワンクリック操作で1ノード以上クラスターを簡単に拡張できます。以下の演習では、仮想デスクトップのプールを拡張して、自由に拡張できるコンピューティングリソースとストレージリソースを活用します。

#. **Citrix Studio > Machine Catalogs** と進み、非永続マシンカタログを右クリックし **Add Machines** を選択する。

   .. figure:: images/1.png

#. **Number of machines to add** に **1** を指定し、 **Next** をクリックする。

#. 既存のOUと naming scheme を確認し、 **Next** をクリックする。

   .. figure:: images/2.png

#. 設定内容を確認し、 **Finish** をクリックする。

#. **Citrix Studio > Delivery Groups**　と進み、非永続デリバリーグループを右クリックし、 **Add Machines** をクリックする。

   .. figure:: images/3.png

#. **number of Machines for this Delivery Group** に **1** を指定し、 **Next > Finish** をクリックする。

   .. note::

      非永続Delivery Groupをダブルクリックして、新しいデスクトップの電源がオフになっていることを確認します。これは :ref:`citrixnpdesktops` 演習で設定した電源管理設定による動作です。

#. 非永続 Delivery Groupの **Power Management** を編集してパワーオン状態の仮想マシン数を増やす。

#. 追加した *Initials*\ **-NP-#** VM　が **Prism** でパワーオンになっていることを確認し、 **Citrix Studio** で登録済みと表示されることを確認する。

   .. figure:: images/4.png

ゴールドイメージの更新
+++++++++++++++++++++++

非永続デスクトップの主な利点の1つは、マスターイメージに変更を加えただけで、更新を多数のシステムに均一に展開できることです。以下の演習では、AHV上のCitrixを使用したプロセスがどれほど高速で簡単であるかを説明します。

#. *Initials*\ **-GoldImage** VMの電源をONにし、コンソール接続する。

#. **GoldImage** VMでアプリケーションをインストールし (例 PuTTY, Atom, 7Zip, など) VMをシャットダウンする。

   .. note::

      ユーザーはローカル管理者グループのメンバーではないため、特定のアプリケーションをインストールできない場合があります。アプリケーションのインストール中にエラーが発生した場合は、Shiftキーを押しながらインストーラーを右クリックし、[ 別のユーザーとして実行 ]を選択します。NTNXLAB \ Administrator資格情報を使用して、インストールを完了します。

   .. figure:: images/5.png

#. *Initials*\ **-GoldImage** VMの電源が落ちたら **Prism Element** で対象VMを選択し、 **Take Snapshot** をクリックする。

   .. note::

      スナップショットは、Citrix AHVプラグインで認識されるように、Prism Elementで取得する必要があります。

   .. figure:: images/6.png

#. スナップショットの **Name** を設定し (例 "*Initials*\ -GoldImage vYYYYMMDD-X - Installed 7zip")  **Submit** をクリックする。

#. **Citrix Studio > Machine Catalogs** で非永続マシンカタログを右クリックし **Update Machines** を選択する。

   .. note::

     永続デスクトップの更新をするためには、別途パッチ管理ツールなどが必要であるため、永続マシンカタログではマシンの更新はできません。

#. Click **Next**.

#. 作成した*Initials*\ **-GoldImage** VM snapshotを選択し、 **Next** をクリックする。

   .. figure:: images/7.png

#. 以下を入力し **Next** をクリックする。

   - **Immediately (shut down and restart the machine now)** を選択
   - **Distribution time** - Update all machines at the same time
   - **Notify users of the update** - Do not send a notification

   .. note::

     これらの選択は、更新をできるだけ迅速に展開することに基づいています。Studioは、管理者がプロアクティブにユーザーに通知し、また数時間にわたる大きなプールのゴールドイメージ展開をずらすことができるようにするなど、ロールアウト戦略に最大限の柔軟性を提供します。

#. 構成を確認し、 **Finish** をクリックする。

   新しいスナップショットを準備するために、新しい準備VMが複製および起動されます。

   .. figure:: images/8.png

#. PreparationVMがシャットダウンされ、削除された後、 **Prism > Tasks** で、VMの電源がOFFにされたタスクを確認する。数分後、VMディスクの更新タスクが表示される。　※これは、プロビジョニングされたVMのクローンディスクをMCSが更新して、新しく準備されたスナップショットを指すようにするタスクです。

   .. figure:: images/9.png

#. Citrix StoreFrontに **NTNXLAB\\operator01** としてログインし、 **Pooled** されたデスクトップを起動し、提示されたデスクトップが更新されたイメージを反映していることを確認する。

   .. figure:: images/10.png

#. **Citrix Studio > Machine Catalogs** にて、非永続マシンカタログを以前のスナップショットへロールバックするためのオプションがあることを確認する。

   .. figure:: images/11.png

お持ち帰り
+++++++++

- 既存のマシンカタログへの容量の追加は迅速に実行できます。ワンクリック操作で物理クラスターを拡張するNutanixの機能と組み合わせると、IT組織は変化するビジネスニーズに非常に迅速に対応できます。

- Nutanix AHVクラスターは、vCenterまたはSCVMMのように、サービスを介することでクローン作成および電源操作がボトルネックになるということがありません。つまり、より多くの同時操作をサポートする機能がクラスターと共にスケールアウトします。このスケールアウトアーキテクチャは、マシンカタログの拡張や更新などのVDI操作を補完します。

- Nutanixは、作成された新しいスナップショットごとに個別のブロックマップ（vDiskを対応するエクステントにマッピングするメタデータ）を作成することで、他のハイパーバイザーで従来見られていた大きなスナップショットチェーンによる追加オーバーヘッドと読み取りレイテンシを排除します。ゴールドイメージ管理は、スナップショットチェーンのパフォーマンスへの影響を軽減する必要がないため、簡素化されます。

- MCSを使用したゴールドイメージのバージョン管理は、スナップショットの命名規則を使用して簡単に実装できます。
