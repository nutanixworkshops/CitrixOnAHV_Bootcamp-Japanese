.. _citrixgettingstarted:

----------------------
はじめに
----------------------

Citrix Apps＆Desktopsをフィーチャーしたエンドユーザーコンピューティングブートキャンプへようこそ。このブートキャンプは、NutanixがCitrixワークロードに理想的なプラットフォームである理由を体験することを目的としています。
Nutanix HCIのスケーラビリティや一貫したパフォーマンスなど、仮想デスクトップにもたらす利点に加えて、以下に記載するような利点を体験頂きます。

- CitrixとライセンスフリーのAHVの統合により、デスクトップ仮想化のための管理しやすいプラットフォームを提供
- デスクトップの大規模なプールへのイメージ更新の適用を含む、高速なデスクトッププロビジョニング
- ユーザーデータ、プロファイル、ユーザーパーソナライゼーションレイヤーを提供するNutanix Filesを使用したファイルサービス
- 仮想デスクトップ環境を保護するためのNutanix Flowによるマイクロセグメンテーション
- Prism Opsによる豊富な監視および自動化機能

Windows Tools VMのデプロイ
+++++++++++++++++++++++++++++++

#. Prism Centralで :fa:`bars` > **仮想インフラ（Virtual Infrastructure）** > **仮想マシン（VMs）** を選択。

#. **仮想マシンを作成（Create VM）** をクリック

#. 以下を入力する:

   - **Name** - *Initials*-WinToolsVM
   - **Description** - (オプション) WinToolsVM.
   - **vCPU(s)** - 2
   - **Number of Cores per vCPU** - 1
   - **Memory** - 4 GiB

   - **+ Add New Disk** をクリック
      - **Type（タイプ）** - DISK
      - **Operation（オペレーション）** - Clone from Image Service（イメージサービスからクローン）
      - **Image（イメージ）** - WinToolsVM.qcow2
      - **Add** をクリック

   - **Add New NIC** をクリック
      - **VLAN Name** - Secondary
      - **Add** をクリック

#. **Save** をクリックしVMを作成する.

#. 作成したVMのチェックボックスをクリックし **Actions（アクション） > Power On（パワーオン）** .

　　起動すると、自動的にSysprepの処理が走り、 **NTNXLAB.local** ドメインに参加し、**NTNXLAB\\Administrator** としてログインします。
