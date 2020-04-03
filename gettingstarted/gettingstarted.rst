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

#. Prism Centralで **:fa:`bars`** > **仮想インフラ（Virtual Infrastructure）** > **仮想マシン（VMs）**を選択。

#. **Create VM**.

#. Select your assigned cluster and click **OK**.

#. Fill out the following fields:

   - **Name** - *Initials*-WinToolsVM
   - **Description** - (Optional) Description for your VM.
   - **vCPU(s)** - 2
   - **Number of Cores per vCPU** - 1
   - **Memory** - 4 GiB

   - Select **+ Add New Disk**
      - **Type** - DISK
      - **Operation** - Clone from Image Service
      - **Image** - WinToolsVM.qcow2
      - Select **Add**

   - Select **Add New NIC**
      - **VLAN Name** - Secondary
      - Select **Add**

#. Click **Save** to create the VM.

#. Select your VM and click **Actions > Power On**.

   Once booted, the VM will automatically complete the Sysprep process, join the **NTNXLAB.local** domain, and log in as the **NTNXLAB\\Administrator** user.
