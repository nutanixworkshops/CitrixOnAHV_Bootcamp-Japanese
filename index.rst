.. title:: Nutanix Citrix on AHV Bootcamp

.. toctree::
  :maxdepth: 2
  :caption: End User Computing - Citrix
  :name: _euccitrix
  :hidden:

  gettingstarted/gettingstarted
  goldimage/goldimage
  pdesktops/pdesktops
  npdesktops/npdesktops
  updatecatalog/updatecatalog
  files/files
  flow_quarantine_vm/flow_quarantine_vm
  flow_secure_desktops/flow_secure_desktops
  prismops/prismops_capacity_lab_ctx/prismops_capacity_lab
  prismops/prismops_rightsize_ctx_lab/prismops_rightsize_euc_lab

.. toctree::
  :maxdepth: 2
  :caption: Appendix
  :name: _appendix
  :hidden:

  tools_vms/windows_tools_vm
  tools_vms/linux_tools_vm
  appendix/glossary

.. _getting_started:

---------------
はじめに
---------------
AHVブートキャンプのNutanix Citrixへようこそ！この演習では、NutanixとCitrixのテクノロジーを利用して、一般的なVDIにおける管理タスクを体験頂きます。

この演習を経て、Nutanix Enterprise Cloudスタックを構成するコアの概念とテクノロジーを理解し、さらにCitrixを利用したVDIの管理手法について学ぶことができます。

アジェンダ
++++++

- イントロダクション
- ゴールデンイメージの準備
- 永続デスクトップ
- 非永続デスクトップ
- Filesの利用
- Flowの利用
- Prism Opsの利用

イントロダクション
+++++++++++++

- 講師紹介

初期セットアップ
+++++++++++++

- 利用する *Passwords* をメモする。
- 仮想デスクトップへログインする。 (接続情報は後述)

環境詳細
+++++++++++++++++++

Nutanixワークショップは、USにあるNutanix Hosted POC環境で実行します。環境には演習に必要なすべての必要なイメージ、ネットワーク、VMがクラスターにプロビジョニングされています。

ネットワーク
..........

Hosted POC クラスターは 以下の命名規則に従い命名されています:

- **Cluster Name** - POC\ *XYZ*
- **Subnet** - 10.**21**.\ *XYZ*\ .0
- **Cluster IP** - 10.**21**.\ *XYZ*\ .37

マーケティングプールからプロビジョニングされた場合は以下:

- **Cluster Name** - MKT\ *XYZ*
- **Subnet** - 10.**20**.\ *XYZ*\ .0
- **Cluster IP** - 10.**20**.\ *XYZ*\ .37

例:

- **Cluster Name** - POC055
- **Subnet** - 10.21.55.0
- **Cluster IP** - 10.21.55.37

環境内には、* XYZ *をサブネットの正しいオクテットに置き換える必要があるVMが存在しています。たとえば：

.. list-table::
   :widths: 25 75
   :header-rows: 1

   * - IP Address
     - 説明
   * - 10.21.\ *XYZ*\ .37
     - Nutanix Cluster 仮想IP
   * - 10.21.\ *XYZ*\ .39
     - **PC** VM IP, Prism Central
   * - 10.21.\ *XYZ*\ .40
     - **DC** VM IP, NTNXLAB.local ドメインコントローラー


各クラスターは、2つのVLANで構成されています:

.. list-table::
  :widths: 25 25 10 40
  :header-rows: 1

  * - Network名
    - Address
    - VLAN
    - DHCP アドレス範囲
  * - Primary
    - 10.21.\ *XYZ*\ .1/25
    - 0
    - 10.21.\ *XYZ*\ .50-10.21.\ *XYZ*\ .124
  * - Secondary
    - 10.21.\ *XYZ*\ .129/25
    - *XYZ1*
    - 10.21.\ *XYZ*\ .132-10.21.\ *XYZ*\ .253

ユーザー名／パスワード
...........

.. 注意::

  *<Cluster Password>* はHosted POC機によって異なります。

.. list-table::
   :widths: 25 35 40
   :header-rows: 1

   * - Credential
     - Username
     - Password
   * - Prism Element
     - admin
     - *<Cluster Password>*
   * - Prism Central
     - admin
     - *<Cluster Password>*
   * - Controller VM
     - nutanix
     - *<Cluster Password>*
   * - Prism Central VM
     - nutanix
     - *<Cluster Password>*

各クラスターには、**NTNXLAB.local**ドメインにADサービスを提供する専用のドメインコントローラーVM **DC**あります。ドメインには、次のユーザーとグループが入力されています。:

.. list-table::
   :widths: 25 35 40
   :header-rows: 1

   * - Group
     - Username(s)
     - Password
   * - Administrators
     - Administrator
     - nutanix/4u
   * - SSP Admins
     - adminuser01-adminuser25
     - nutanix/4u
   * - SSP Developers
     - devuser01-devuser25
     - nutanix/4u
   * - SSP Consumers
     - consumer01-consumer25
     - nutanix/4u
   * - SSP Operators
     - operator01-operator25
     - nutanix/4u
   * - SSP Custom
     - custom01-custom25
     - nutanix/4u
   * - Bootcamp Users
     - user01-user25
     - nutanix/4u

環境アクセス方法
+++++++++++++++++++

Nutanix Hosted POC 環境には以下の方法で接続できます。:

ユーザ名、パスワード
...........................

PHX クラスター:
**Username:** PHX-POCxxx-User01 〜 PHX-POCxxx-User20, **Password:** *<講師から提供>*

RTP クラスター:
**Username:** RTP-POCxxx-User01 〜 RTP-POCxxx-User20, **Password:** *<講師から提供>*

Frame VDI
.........

ログイン: https://frame.nutanix.com/x/labs

上記 **Username**, **Password** を入力

Parallels VDI
.................

PHX クラスター: https://xld-uswest1.nutanix.com

RTP クラスター: https://xld-useast1.nutanix.com

上記 **Username**, **Password** を入力

Nutanix Version Info
++++++++++++++++++++

- **AHV Version** - AHV 20170830.337
- **AOS Version** - 5.11.2.3
- **PC Version** - 5.11.2.1
