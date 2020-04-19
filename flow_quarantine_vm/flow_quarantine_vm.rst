.. _euccitrixflow_quarantine_vm:

-------------------------------
Flowによるデスクトップの隔離（Quarantine)
-------------------------------

隔離（Quarantine）は制限されたポリシーにVMを配置し、管理者にすべてのネットワークトラフィックをブロックするか、トラフィックの限られたサブセットを許可するオプションを与えます。Strict quarantine（厳密な検疫）はすべての通信からVMをブロックしますが、Forensic quarantine（フォレンジック検疫）は着信および発信トラフィックの事前定義されたリストを許可します。この機能は、VMがマルウェアの影響を受けている仮想デスクトップ環境で非常に役立ちます。

**このラボでは、デスクトップVMを検疫に配置し、VMの動作を観察します。また、隔離ポリシー内の構成可能なオプションを検査して、感染したVMのトラブルシューティングをシミュレートします。**

SecOps VMのカテゴライズ
++++++++++++++++++++++++++

#. **Prism Central** で :fa:`bars` **> 仮想インフラ（Virtual Infrastructure） > カテゴリ（Categories）** と選択する。

#. **AppType** のチェックボックスを選択し、 **アクション（Actions） > Update** をクリックする。

   .. figure:: images/12.png

#. 最後の値の右にある :fa:`plus-circle` アイコンをクリックする。

#. *Initials*-**SecOps** と入力する。

   .. figure:: images/13.png

#. **保存（Save）** をクリックする。

#. **Prism Central** で :fa:`bars` **> 仮想インフラ（Virtual Infrastructure） > 仮想マシン（VMs）** を選択する。

#. *Initials*\ **WinToolsVM** のチェックボックスにチェックを入れ、 **アクション（Actions） > カテゴリの管理（Manage Categories）** とクリックする。

   .. figure:: images/14.png

#. **AppType:**\ *Initials*-**SecOps** を検索し、 **保存（Save）** をクリックし、VMにカテゴリを割り当てる。

   .. figure:: images/15.png

デスクトップへのアクセスおよび検疫
+++++++++++++++++++++++++++++++++++++++

#. *Initials*\ -**WinToolsVM** にてブラウザを開き、 http://ddc.ntnxlab.local/Citrix/NTNXLABWeb へアクセスし、Citrix StoreFront serverに接続する。

#. 以下の資格情報で **Log On** する。:

   - **Username** - NTNXLAB\\devuser01
   - **Password** - nutanix/4u

#. **Desktops** タブで **Personal Win10 Desktop** を選択し、セッションを開始する。

#. また、 *Initials*\ **WinToolsVM** で **Command Prompt** を起動し、 ``ping -t XYZ-PD-1-VM-IP`` を実行しクライアントから仮想デスクトップへの疎通を確認する。（隔離後にどうなるか確認するため）

#. **Prism Central > 仮想インフラ（Virtual Infrastructure） > 仮想マシン（VMs）** と進み、 *Initials*\ **-PD-1** と *Initials*\ **-PD-2** のチェックボックスにチェックをいれる。

#. **アクション（Actions） > Quarantine VMs** をクリックする。

   .. figure:: images/1.png

#. **フォレンジック（Forensic）** をクリックし、 **隔離（Quarantine）** をクリックする。

   Windows Tools VMから実行していたpingはどうなっていますか？

カスタムQuarantine Policyの作成
+++++++++++++++++++++++++++++++++++

#. **Prism Central** にて :fa:`bars` **> ポリシー（Policies） > セキュリティポリシー（Security Policies） > Quarantine** と進み、検疫中のVMを確認する。

#. **更新（Update）** をクリックし、Quarantine policyを編集する。

   このFlowポリシーの機能を説明するために、Windows Tools VMを「フォレンジックツール」として追加します。本番環境では、隔離されたVMへのインバウンドアクセスを許可されたVMを使用して、Kali LinuxやSANS SIFTなどのセキュリティおよびフォレンジックスイートを実行できます。

#. **次へ（Next）** をクリックし、ポリシー編集画面に移動する。

#. **インバウンド（Inbound）** にて **+ 移行元を追加（+ Add Source）** をクリックする。

#. 以下を入力する。:

   - **Add source by:** - **Category（カテゴリ）** を選択
   - **AppType:**\ *Initials*-**SecOps** を指定

   .. figure:: images/16.png

#. **追加（Add）** をクリックする。

   このソースはどのターゲットに接続できますか？ Forensic(フォレンジック)とStrict(厳格)の隔離モードの違いは何ですか？

   VMを **Strict** （厳格）な検疫ポリシーに追加すると、VMへのすべてのインバウンドおよびアウトバウンド通信が無効になることに注意してください。 **Strict** （厳格）なポリシーは、ネットワーク上の環境への脅威となる仮想マシンに適用します。

#. **Quarantine: フォレンジック（Forensic）** の左側にある :fa:`plus-circle` アイコンをクリックし、受信ルールを追加する。

#. **保存（Save）** をクリックし、SecOpsカテゴリのVMと **Quarantine: フォレンジック（Forensic）** カテゴリのVMで、全てのポートのプロトコルを許可する。

   .. figure:: images/17.png

#. **次へ（Next）** をクリックし、 **ここで適用する（Apply Now）** をクリックしてポリシーを適用する。

   Windows Tools VMから実行していたpingはどうなっていますか？

#. Prism CentralでVMを選択し、 **アクション（Actions） > Unquarantine VMs** をクリックして、 **Quarantine: フォレンジック（Forensic）** カテゴリからデスクトップVMを削除できます。


お持ち帰り
+++++++++

- この演習では、Flowを使用して、厳密でフォレンジックな検疫ポリシーの2つのモードを使用してデスクトップVMを検疫しました。
- 検疫ポリシーは、アプリケーションポリシーよりも高い優先度で評価されます。検疫ポリシーは、アプリケーションポリシーで許可されないトラフィックをブロックできます。
- フォレンジックモードは、VMが隔離されている間、隔離されたVMへの制限付きアクセスを許可するためのキーです。
