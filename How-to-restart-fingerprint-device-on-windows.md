# Windows の指紋認証デバイスが動かなくなったときに、そのデバイスだけ再起動する BAT ファイル

Windows PC を使っている際に、指紋認証デバイスでのログインができなくなることがある。
こういった場合は PC を再起動する、もしくは、指紋認証デバイスを一度デバイスマネージャからアンインストールして、デバイスを再スキャンすることで、利用できるようになる。

再起動するのも、GUI でデバイスマネージャを操作するのも面倒なので、デバイスのアンインストール、再インストールを BAT ファイルで実行できるようにした。

以下の手順で実施する。

1. 指紋認証デバイスのインスタンスIDを取得
2. この ID を埋め込んだスクリプトを作る
3. 管理者権限で実行するように設定する

## インスタンス ID の取得方法
コマンドプロンプトにて、
```
%WINDIR%\System32\pnputil.exe /enum-devices /class Biometrics
```
とすることで、インスタンス ID が得られる。手元で簡単に確認している範囲では変化することはなかった。
仮に変化する要素があった場合、もうちょっと頑張ってスクリプトを書く必要があるかも。

仮に変化した場合であっても、 VID と PID を含んでいるので、意図しないデバイスを削除してしまうことは無いと思われる。指紋認証デバイスを複数使っているような、酔狂な環境だとダメかもだが。

実行例
```
C:\Users\kokubo>%WINDIR%\System32\pnputil.exe /enum-devices /class Biometric
Microsoft PnP ユーティリティ

インスタンス ID:                USB\VID_27C6&PID_521D\6&13c452c7&0&3
デバイスの説明:         Goodix fingerprint
クラス名:                 Biometric
クラス GUID:                 {53d29ef7-377c-4d14-864b-eb3a85769359}
製造元の名前:          Goodix
状態:                     開始
ドライバー名:                oem13.inf
```


## BAT ファイル

以下を .bat の拡張子で保存する。
```
%WINDIR%\System32\pnputil.exe /remove-device "USB\VID_27C6&PID_521D\6&13c452c7&0&3"
@%WINDIR%\System32\timeout.exe 1
%WINDIR%\System32\pnputil.exe /scan-devices
@%WINDIR%\System32\timeout.exe 1
```

実行例として、管理者権限でのコマンドプロンプトから起動したログを以下に示す。
```
C:\e\bin\utils>reset-fingerprint-device.bat

C:\e\bin\utils>C:\WINDOWS\System32\pnputil.exe /remove-device "USB\VID_27C6&PID_521D\6&13c452c7&0&3"
Microsoft PnP ユーティリティ

デバイスを削除しています:          USB\VID_27C6&PID_521D\6&13c452c7&0&3
デバイスが正常に削除されました。


0 秒待っています。続行するには何かキーを押してください ...

C:\e\bin\utils>C:\WINDOWS\System32\pnputil.exe /scan-devices
Microsoft PnP ユーティリティ

デバイス ハードウェアの変更をスキャンしています。
スキャンが完了しました。


0 秒待っています。続行するには何かキーを押してください ...

C:\e\bin\utils>
```

## 管理者権限での実行設定
Explorer で BAT ファイルのショートカットを作成することで、
そのショートカットのプロパティ -> 「プロパティ」タブ ->
「詳細設定」ボタン -> 「管理者として実行」チェックボックス という設定項目にアクセスできる。

このチェックを入れると、当該のショートカットを開くだけで、管理者権限の問い合わせダイアログが出てくるようになる。

## 実行
これで、
1. BAT ファイルを開く
2. 管理者権限での実行を承認する

だけで、指紋認証デバイスをリセットできるようになる。
