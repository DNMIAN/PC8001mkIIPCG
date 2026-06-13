# PCG8200のようなもの PC-8001mkII専用 漢字ROM付き<br>
<p align="center">
  <img src="/jpg/0101.jpg" width="30%">
  <img src="/jpg/0102.jpg" width="30%">
  <img src="/jpg/0103.jpg" width="30%">
</p>
<br>
実機でPCG用のソフトを動かしたい！<br>
クローンを製作されている方はいらっしゃいますが、pico2の実験も兼ねて作ってみました。<br>
<p align="left">
  <img src="/jpg/0201.jpg" width="33%">
</p>
PC-8001mkIIのキャラクターROMはソケット実装で漢字ROMコネクタの近くにあります。<br>
幸いにも、漢字ROMコネクタは漢字ROM専用ではなく拡張スロット直結になっていました。<br>
これを利用してコンパクトに実装出来ましたが、引き換えにレイアウトの都合でPC-8001mkII専用になります。<br>
<br>
仕様はインターネットで収集した情報と手持ちソフトの現物合わせで製作しています。<br>
そのため全てのソフトについての動作保証はできません。<br>
WeactRP2350Bの5Vトレラントを利用してGPIOをバスへ直結しています。<br>
RP2350Bはクロック200MHz（オーバークロック）、コア電圧は1.1V(デフォルト)で動作させます。<br>
180MHz程度でも問題は無いのですがマージンを取って200MHzにしています。<br>
本ボードのIOアクセスを多く実行するとテキスト文字がちらつきます。<br>
気になるかは主観ですが、私は許容範囲ではないかと思います。<br>
安定動作と引き換えにクロックを上げれば少しは改善されます。全く非推奨です。<br>
開発に使用した個体で、正規実装(PC本体ケースを閉めている)、300MHz、気温25℃、3時間程度の連続稼働（画面表示＋音出力）を試した結果、コア発熱と動作に問題は見当たりませんでした。<br>
<br>
 
### 認識している相違点など<br>
・PCG定義時の「ROMコピー」機能を実装しています。 (PCG8100の機能)<br>
・PCGとは別にANKカタカナとひらがなの切替ができます。(PC-8001mkIISRのような機能)<br>
・i8253の可聴域カウンタ値を0x0101～0xfffeとしています。（仕様不明、確認したソフトでの現物合わせです）<br>
・i8253の「カウントモード」についてはモード3(方形波)のみ確認しています。<br>
&ensp;一応、全モード実装していますが未確認のため実機と同じ動作をしない可能性が高いです。<br>
・CGROM以外のフォントは実装や字体が実機と異なります。<br>
<br>
&ensp;CGROM(D2316EC)以外のフォントは[美咲フォント](https://littlelimit.net/misaki.htm)および[東雲フォント](http://openlab.ring.gr.jp/efont/shinonome/)から作成しています。<br>
&ensp;おそらく実機は78JIS、使用したものは新しい規格なので一部の実装に差異があります。また字体も違います。<br>
<br>
&ensp;フォント関連は実機から吸い出したものに変更可能です。変更可能なデータとサイズは以下となります。<br>
&ensp;&ensp;・第1水準漢字ROM(PC-8001mkII-01, PC-8801-01K等） サイズ128KBytes<br>
&ensp;&ensp;・第2水準漢字ROM(PC-8801-20等のデータを変換する、[こちら](https://web.archive.org/web/20251004032226/http://www5f.biglobe.ne.jp/~apaslothy/cgi_bin/adiary/adiary.cgi/068)に情報があります） サイズ128KBytes<br>
&ensp;&ensp;・ひらがなROM(PC-8001mkIISRのひらがなANKコードA0h～DFh等) サイズ512Bytes<br>
&ensp;&ensp;・電源投入時のPCG FChキャラクタ(RAM0の7Ch) サイズ8Bytes<br>
<br>
&ensp;&ensp;&ensp;FChキャラクタとは、BASICのOkプロンプトの右に表示されるキャラクタです。<br>
&ensp;&ensp;&ensp;電源投入時のPCGパターンはROMと同じにしてあるためFChはスペースです、PCGオンにしても見えません。<br>
&ensp;&ensp;&ensp;そこでPCGオン状態を認識しやすいようにFChだけ別途指定出来るようにしました。<br>
<br>
&ensp;&ensp;&ensp;作成したデータはpicotoolで書き込みます。<br>
&ensp;&ensp;&ensp;&ensp;picotool.exe load -v -x kanji1.rom -t bin -o 0x10200000<br>
&ensp;&ensp;&ensp;&ensp;picotool.exe load -v -x PC-8801-20.mod.rom -t bin -o 0x10220000<br>
&ensp;&ensp;&ensp;&ensp;picotool.exe load -v -x D2316EC.rom -t bin -o 0x10240000<br>
&ensp;&ensp;&ensp;&ensp;picotool.exe load -v -x hirafont.rom -t bin -o 0x10240800<br>
&ensp;&ensp;&ensp;&ensp;picotool.exe load -v -x fc.rom -t bin -o 0x10240a00<br>
<br>
&ensp;&ensp;&ensp;&ensp;※繋げて一発で書き込む方が楽かもしれません<br>
&ensp;&ensp;&ensp;&ensp;copy /b kanji1.rom+PC-8801-20.mod.rom+D2316EC.rom+hirafont.rom+fc.rom merge.rom<br>
&ensp;&ensp;&ensp;&ensp;picotool.exe load -v -x merge.rom -t bin -o 0x10200000<br>
<br>
 
### 実装前準備
必要なファームウェアとデータはフラッシュメモリへ書き込み済みです。<br>
デフォルト使用の場合は書き込みは不要です。<br>
キャラクターROMソケットに接続するためピンをボード裏のソケットに挿し込みます。ピンの高さがまばらでも良いです。<br>
ピンにICソケットを挿し込みます。奥まで挿し込むことで結果的にピンの高さは揃います。<br>
ICソケットの足が曲がってしまった場合は、適当に直しておいてください。<br>
<p align="left">
  <img src="/jpg/0301.jpg" width="33%">
  <img src="/jpg/0302.jpg" width="33%">
</p>
<br>

### 実装方法<br>
本体カバーを空け（裏面に6箇所ネジがあります）キーボードを手前にずらし、電源ユニットのネジを４箇所外します。<br>
<p align="left">
  <img src="/jpg/0303.jpg" width="33%">
</p>
電源ユニットを手前にずらし、実機のキャラクターROM(D2316EC)を抜きます。<br>
抜いたキャラクターROMは使用しないので保管してください。<br>
専用工具で抜くことをお勧めします。ドライバーなどを使う場合はソケット下のパターンを切断しないように気を付けてください。<br>
<p align="left">
  <img src="/jpg/0304.jpg" width="33%">
  <img src="/jpg/0305.jpg" width="33%">
</p>
本ボードを漢字ROMコネクタとキャラクターROMソケットに合わせて挿し込みます。<br>
音声出力はミニジャックをアンプのライン入力などへ接続します。<br>
<p align="left">
  <img src="/jpg/0306.jpg" width="33%">
</p>
本ボード上のSPEAKER端子にスピーカーを接続することもできます。<br>

私は、[このスピーカー](https://akizukidenshi.com/catalog/g/g103285/)が安くておすすめです。<br>
ただし厚みがあるので本体内に実装できる場所が拡張スロット内くらいしかありません。<br>
拡張スロットにボードを挿していたら無理かもしれません。<br>
キーボードの下も無理でした。補強金具を何とかすれば行けるかもしれませんが。<br>

ちなみに、[このスピーカー](https://akizukidenshi.com/catalog/g/g112495/)では音量が足りませんでした。<br>
なおライン出力とSPEAKER端子は同時使用可能です。<br>
<br>

### 実装後の確認<br>
本体ケースを閉める前にPCの電源を入れて、BASIC起動画面などで文字がきちんと表示されているか確認してください。<br>
文字がおかしい場合は漢字ROMコネクタ、キャラクターROMソケットにきちんと挿さっているか再確認をしてください。<br>
またWeactRP2350Bが通電しているか確認してください。(赤LEDが点灯)<br>
BASICから out 3,8 とし、Okの横に文字（デフォルトの場合は「です」）が表示されるか確認してください。<br>
out 12,160:out 12,59 とし、ド音が出力されることを確認してください。<br>
ここまで確認できれば実装は問題ないと思います。<br>
<p align="left">
  <img src="/jpg/0307.jpg" width="33%">
</p>
<br>

### 熱対策ボードについて<br>
電源からの熱を反射してRP2350の熱対策になるらしい板を載せます。<br>
別に載せなくても良いです。効果があるか分かりません。AIの受け売りで作りました。<br>
先の300MHz耐久試験時は載せていませんでした。<br>
<p align="left">
  <img src="/jpg/0308.jpg" width="33%">
</p>
<br>

### 補足：PCGオンオフについて<br>
PCG8200はソフトでオンオフ切替をします。<br>
PCG対応ソフトは out 3,8 としてから実行します。PCをリセットしても状態は継続します。<br>
ソフトによっては自動的にPCGオンになるものもありました。<br>
<br>
&ensp;IOアドレス 03h OUT PCGセレクト<br>
&ensp;&ensp;k--rhhll<br>
&ensp;&ensp;&ensp;k  カタ/ひらモード =0:カタカナ =1:ひらがな（PCGオフ・ROMコピー時に参照）※本ボードのみ<br>
&ensp;&ensp;&ensp;r  PCG定義RAM =0:RAM0 =1:RAM1<br>
&ensp;&ensp;&ensp;hh 表示するHI(80h～FFh)のバンク<br>
&ensp;&ensp;&ensp;ll 表示するLO(00h～7Fh)のバンク<br>
<br>
&ensp;&ensp;&ensp;バンクは =00,01:CGROM(PCGオフ) =10:RAM0(PCGオン) =11:RAM1(PCGオン)<br>
&ensp;&ensp;&ensp;電源投入時は定義RAM0でHI,LO共にCGROM(PCGオフ)<br>
&ensp;&ensp;&ensp;PCG定義は表示するバンク状態には影響されない<br>
&ensp;&ensp;&ensp;BASICから out 3,8 でPCG8100互換でPCGオン、out 3,0 でPCGオフ<br>
<br>
カタ/ひらモード切替でPCGを使わずにひらがな表示<br>
<p align="left">
  <img src="/jpg/0401.jpg" width="33%">
</p>
<br>

### 本ボード独自のポート拡張について<br>
以下のIOを拡張しています。<br>
&ensp;IOアドレス EAh IN  コア電圧設定(単位10mV)読み出し<br>
&ensp;IOアドレス EBh IN  現在クロック(単位MHz)読み出し<br>
&ensp;IOアドレス 04h OUT 音量変更<br>
&ensp;&ensp;80h+音量(0～127)でoutし、音量(0～127)でoutすると設定します、電源断で設定は消えます<br>
&ensp;&ensp;BASICから out 4,&h80+127:out 4,127 で音量が127(最大)<br>
<br>

### その他<br>
通常使用では不要です、使用する場合はハード追加が必要なものもあります。<br>
#### ・緊急停止スイッチ<br>
ボード上のSTOPジャンパーをショートすると、RP2350Bが強制停止します。<br>
本ボードが動作不良となった場合、テキスト文字が全て■になることが予想されます。<br>
STOPジャンパーをショート後、オープンにすると本ボードは再スタートします。（電源投入時の状態）<br>
ハード故障の場合は再スタートできませんが、ソフトのデッドロックならテキスト文字は戻ります。<br>
このような状態になった場合は、継続使用はせず一度PCの電源を切にしてください。<br>
今まで一度もなったことはありませんが、突然テキスト文字が全て■になることでそのまま電源を切るのが困る方、文字が見えない状態でリカバリが出来ない方はSTOPジャンパーへスイッチなどを接続し、緊急時に操作可能としておくと少し安心かもです。<br>
#### ・CGROMの使用（パターンミスのためD2の実装に工夫が必要、V2.1基板で修正予定）<br>
緊急停止スイッチのショート時に、CGROMを使って文字表示をします。<br>
R10(10kΩ),D2(1N4148など),24ピンICソケット(裏面)を実装し、ICソケットにD2316ECを挿します。<br>
実装時に1mm程度の隙間はありますが、念のためD2316ECにポリイミドテープなどを貼って絶縁してください。<br>
<p align="left">
  <img src="/jpg/0501.jpg" width="33%">
  <img src="/jpg/0502.jpg" width="33%">
</p>

#### ・クロックおよびコア電圧の変更<br>
フラッシュメモリの設定値を書き換えることで可能です。全く非推奨です。<br>
config.binを書き換えてpicotoolで書き込みます。<br>
BASICから ?inp(&hea) でコア電圧設定、?inp(&heb) で現在クロックを確認できます。<br>
<br>
&ensp;config.bin サイズ16Bytes<br>
&ensp;&ensp;0000 : 63 6F 6E 66 69 67 cc cc cc rr rr rr vv 00 00 00<br>
<br>
&ensp;&ensp;クロックを(200 + cc)MHzとしてcc値をMHzで設定する、デフォルト0、3か所全て同じ値、00～64h(=100)以外は無視する<br>
&ensp;&ensp;コア電圧rrを設定する、デフォルト0(1.10V)、=1で1.15V、=2で1.20V、3か所全て同じ値、1,2以外は無視する<br>
<br>
&ensp;picotool.exe load -v -x config.bin -t bin -o 0x102fff00<br>
<br>
#### ・デフォルト音量の変更<br>
接続したスピーカーのワット数により音割れする場合、音量を絞ることで改善するかもしれません。<br>
前述のconfig.binを書き換えます。<br>
&ensp;音量vvを設定する、デフォルト値120、0～7Fh(=127)以外は無視する<br>
<br>
#### ・PC-8001mkII_SD互換<br>
yanatakaさんの[PC-8001mk2_SD](https://github.com/yanataka60/PC-8001mk2_SD)を実装します。<br>
PC-8001mkII本体に挿す拡張ROMは別途必要です。これについてはPC-8001mk2_SDの解説を参照してください。<br>
<br>
以下の注意事項です。<br>
&ensp;・SDカードの代わりに、USBメモリ(FATまたはFAT32)を使用します。16GB(FAT32)でしか検証していません。<br>
&ensp;~~・killコマンドは正常を返すとPC側のコマンドが復帰しないので、削除成功しても常にエラーを返します。~~ <br>
&ensp;&ensp;修正しました。<br>
&ensp;・書き込み系はSAVEコマンドとモニタのWコマンドしか動作確認していません。<br>
&ensp;・PC側とのシーケンスが崩れた場合は永久waitになる場合があります。STOPキーを押しながらリセットしてください。<br>
&ensp;~~・現時点のファームではメガCD版シルフィードのデモやBad Apple全編などのロードが多いcmtファイルは、<br>
&ensp;&ensp;途中で読み込みエラーになります。後で時間がある時に調べます。~~ <br>
&ensp;&ensp;修正しました。またマシン語ロード時のみcmtファイルの最後まで到達後のロードは先頭に巻き戻し（cmtファイル自動再オープン）を試みるようにしました。（シルフィードデモは永久ループします）<br>
<br>
使用するにはWeactRP2350Bのファームウェアを更新してください。<br>
更新方法は「pico ファームウェア更新方法」などで調べてください。<br>
PC-8001mk2_SD対応のファームウェア使用中の場合、PC-8001mk2_SDと同時に実装しないでください、最悪壊れます。<br>
本機能を使用しない場合は、通常ファームウェアを使用してください。<br>
<br>
&ensp;&ensp;ext8200sd.uf2 : PC-8001mk2_SD対応のファームウェア<br>
&ensp;&ensp;ext8200.uf2   : 通常ファームウェア<br>
