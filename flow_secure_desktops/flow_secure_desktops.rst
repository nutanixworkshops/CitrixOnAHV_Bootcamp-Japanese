.. _ctxflow_secure_desktops:

---------------------------
Flowによるデスクトップの保護
---------------------------

Nutanix AHVで仮想デスクトップワークロードを実行するもう1つの利点は、Flowでネイティブのマイクロセグメンテーション機能を利用できることです。Flowは、VM間の強力なファイアウォールルールをグラフィカルに監視およびモデリングし、受信および送信アクセスを制御する機能を提供します。これは、Webサーバーやデスクトップなど、攻撃の阻止においてVMからVMへのトラフィックの拡散を防ぐことが重要なアプリケーションに最適です。

**この演習では、デスクトップVMをアプリケーションポリシーに配置して、同一Tier内の通信を制限します。デスクトップへの通信は可能ですが、デスクトップ間のトラフィックはブロックされることを確認します。**

デスクトップVMのカテゴライズ
++++++++++++++++++++++++++++

#. **Prism Central** で :fa:`bars` **> 仮想インフラ（Virtual Infrastructure） > カテゴリ（Categories）** を選択する。

#. **AppType** を選択し、 **アクション（Actions） > Update** をクリックする。

   .. figure:: images/1.png

#. 最後のカテゴリの側にある :fa:`plus-circle` をクリックしカテゴリを追加する。

#. *Initials*-**Desktops** を入力する。

   .. figure:: images/2.png

#. **保存（Save）** をクリックする。

#. **AppTier** のチェックボックスを選択し、 **アクション（Actions） > Update** をクリックする。

#. 最後のカテゴリの側にある :fa:`plus-circle` をクリックしカテゴリを追加する。

#. *Initials*-**PD** を入力する。

#. :fa:`plus-circle` を再度クリックし、 *Initials*-**NPD** を追加する。

   .. figure:: images/3.png

#. **保存（Save）** をクリックする。

#. **Prism Central** にて :fa:`bars` **> 仮想インフラ（Virtual Infrastructure） > 仮想マシン（VMs）** と進む。

#. 永続デスクトップである *Initials*\ -**PD** をチェックボックスで選択し、 **アクション（Actions） > カテゴリの管理（Manage Categories）** と進む。

   .. figure:: images/4.png

#. 検索バーにて **AppType:**\ *Initials*-**Desktops** を検索する。

#. 最後の項目の側にある :fa:`plus-circle` アイコンをクリックし、 **AppTier:**\ *Initials*-**PD** を選択し、 **保存（Save）** をクリックする。

   .. figure:: images/5.png

#. 先の手順を繰り返し、**AppType:**\ *Initials*-**Desktops** および **AppTier:**\ *Initials*-**NPD** カテゴリを非永続デスクトップに割り当てる。

セキュリティポリシーの作成
++++++++++++++++++++++++++++++++++

#. **Prism Central** で :fa:`bars` **> ポリシー（Policies） > セキュリティポリシー（Security Policies）** と進む。

#. **セキュリティポリシーの作成（Create Security Policy） > セキュアアプリケーション（アプリケーションポリシー）（Secure Applications (App Policy)） > 作成（Create）** と進む。

#. 以下を入力する。:

   - **名前（Name）** - *Initials*-Desktops
   - **目的（Purpose）** - Restrict unnecessary traffic between desktops
   - **このアプリを保護（Secure this app）** - AppType: *Initials*-Desktops
   - **カテゴリ別にアプリタイプにフィルターをかけます（Filter the app type by category）** は選択 **しない** 。

   .. figure:: images/6.png

#. **次へ（Next）** をクリックする。

#. ナビゲートウィンドウが表示されたら、 **分かりました（OK, Got it!）** をクリックする。

#. より詳細にセキュリティポリシーを設定するために **代わりにアプリ階層にルールを設定します（Set rules on App Tiers, instead）** をクリックする。これによりAppType全体ではなく、カテゴリ単位で設定することができる。

   .. figure:: images/7.png

#. **+ 階層を追加（Add Tier）** をクリックする。

#. ドロップダウンから、 **AppTier:**\ *Initials*-**PD** を選択する。

#. **AppTier:**\ *Initials*-**NPD** に対して手順7-8を再度実施する。

   .. figure:: images/8.png

   次に、アプリケーションとの通信を許可するソースを制御するインバウンドルールを定義します。今回はすべての受信トラフィックを許可します。

#. 画面左側にある **インバウンド（Inbound）** にて、 **ホワイトリストのみ（Whitelist Only）** から **全て許可（Allow All）** に変更する。

   .. figure:: images/9.png

#. 画面右側の **アウトバウンド（Outbound）** も同様に **全て許可（Allow All）** に変更する。

#. デスクトップ内の通信制御を定義するために、 **アプリ内でルールを設定（Set Rules within App）** をクリックする。

   .. figure:: images/10.png

#. **AppTier:**\ *Initials*-**PD** を選択し、同一Tier内のVM間通信を制限するために **No** を選択する。これにより、永続デスクトップ間の通信を制限することができる。

   .. figure:: images/11.png

#. **AppTier:**\ *Initials*-**PD** を選択した状態で **AppTier:**\ *Initials*-**NPD** の右側にある :fa:`plus-circle` アイコンをクリックし、ルールを作成する。

#. 以下を入力し、非永続デスクトップと、永続デスクトップ間のTCP port **7680** を許可し、peer-to-peer のWindows update通信を許可する。:

   - **Protocol** - TCP
   - **Ports** - 7680

   .. figure:: images/12.png

#. **保存（Save）** をクリックする。

#. **AppTier:**\ *Initials*-**NPD** を選択し、同一Tier内のVM間通信を制限するために **No** を選択する。

#. **次へ（Next）** をクリックし、設定したsecurity policyのルールを確認する。

#. **保存とモニター（Save and Monitor）** をクリックし、ポリシーを保存する。

セキュリティールールの検証
++++++++++++++++++++++++++

#. Prism Central にて永続デスクトップVMのIPアドレスを確認し、控えておきます。

#. *Initials*\ -**WinToolsVM** にてブラウザを起動し、http://ddc.ntnxlab.local/Citrix/NTNXLABWeb にアクセスしCitrix StoreFront serverに接続する。

#. 以下の資格情報を入力し **Log On** をクリックする。:

   - **Username** - NTNXLAB\\devuser01
   - **Password** - nutanix/4u

#. **Desktops** タブを選択し、 **Personal Win10 Desktop** をクリックしてセッションを開始する。。

#. 永続デスクトップにて **Command Prompt** を起動し、 ``ping -t XYZ-PD-VM-IP`` を実行し、永続デスクトップ間の通信を確認する。

   .. figure:: images/13.png

   疎通は通りますか？またなぜその状態になるでしょうか？

#. **Prism Central > ポリシー（Policies） > セキュリティポリシー（Security Policies）** と進み、 *Initials*\ **-Desktops** ポリシーを選択する。

#. **アクション（Actions） > 適用（Apply）** をクリックする。

   .. figure:: images/14.png

#. ポリシーを適用するために **APPLY** 、 **OK** をクリックする。

   先ほど実行した疎通はどうなりましたか？

お持ち帰り
+++++++++

- この演習では、Flowを使用してデスクトップ間のトラフィックをブロックし、マルウェアの環境内拡散を防ぎました。
- Monitor（監視）モードは定義されたアプリケーションへのトラフィックを視覚化するために使用され、Apply（適用）モードはポリシーを実行します。
- アプリケーションポリシーは、従来のアプリケーションと同様にデスクトップを保護するために使用できます。
