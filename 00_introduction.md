# 00 導入

これは京都大学理学部化学教室 理論化学分科でのB4M1ゼミ向けに作成した資料です。
主に研究室に入ってきたばかりの4回生向けに、量子化学計算の雰囲気を勉強してもらうためのチュートリアルとなっています。
PySCF向けに作成したものは[こちら](https://github.com/YoshioNishimoto/sandbox/wiki)。

## 量子化学計算

まずは量子化学計算で主に行うことを説明しておきます。
おおざっぱに言えば、`量子力学・量子化学的な方法を用いて分子のエネルギー、構造、性質を計算することで、
化学的な現象を再現・予測すること`が目的となります。
この計算のベースとなっているのは、基本的には量子化学の授業で勉強した事項となっています。
Hartree–Fock法[^1]、Full CI (configuration interaction)法、ヒュッケル法あたりは量子化学の授業でも出てきたと思います。
このチュートリアルでは、これら（？）の方法を使って、実際に分子のエネルギーを計算します。

ヒュッケル法でやったとおり、かなり大胆な近似をしているにもかかわらず、エネルギーを計算するためには行列の対角化が必要でした。
このため、Hartree–Fock法やFull CI法の計算の大変さは、ヒュッケル法の比ではありませんので、普通の電卓を使ったとしても手計算で現実的な分子を扱うことは不可能と言っても良いでしょう。
そこで、量子化学計算はコンピュータおよび専用のプログラムを用いて行います。
しかし、量子化学計算を行うプログラムを自分で作成するというのは、理論を勉強するのも大変ですが、勉強した数式をプログラムにするのも大変です。
幸いにも量子化学計算には過去数十年の蓄積があり、プログラムを一から作成する必要性は、現代ではそれほどありません[^2]。

そのような量子化学計算プログラムとして代表的なものは[Gaussian](https://gaussian.com/)、[GAMESS(-US)](https://www.msg.chem.iastate.edu/gamess/)、[ORCA](https://orcaforum.kofo.mpg.de/)あたりが有名でしょう。
このうち、Gaussianは有料ですが、現代では実験化学者でもGaussianを用いて量子化学計算を行い、反応機構の予測に役立てるということは日常的に行われています。
このチュートリアルでは、[OpenMolcas](https://gitlab.com/Molcas/OpenMolcas)というオープンソースの量子化学計算プログラムを用います。

## OpenMolcas

OpenMolcasの説明を少ししておきます。
上に挙げたGaussian等は、かなり汎用性の高いパッケージとなっています。
特にGaussianは有機化学者が使うことを前提としたプログラム設計がされており、マイルドな言い方をすれば誰でも扱うことができるようになっています。

一方、OpenMolcasは多配置計算と多参照摂動理論の計算に特化したプログラムとなっています。
これらの手法は扱う分子がどのような特徴を持っているかを把握する必要があり、またどのような計算をしたいかも把握しておかなければなりません。
このため、より「専門的な」手法を扱うことに特化したプログラムです。
もともとはMOLCASという名前でスウェーデンのルンド大学（Björn Roosグループ）で開発されたものです。
長いあいだ有料のソフトウェアでしたが、Wikipediaによると2017年から大半のコードがOpenMolcasとしてオープンソースで利用できるようになりました。

というわけで、OpenMolcasのインストールをしたいのですが、あまり簡単ではありません。
今時のアプリ（？）のように、何回かクリック・タップしてやればインストールできて、インストールすればあとは簡単に実行できるかと言えばそうではありません。
OpenMolcasは自分でソースコードをコンパイルしなければなりませんし、インストールした後はCUI (command user interface)でプログラムを実行します。
そもそも、パソコンを一般的に使っている限りでは、CUIを使う環境がないという場合もあります。
このため、以下のような手順でインストールをしていきます。

- CUI環境の準備
- Gitによるソースコードの取得
- OpenMolcasのコンパイル
- OpenMolcasのコンパイル（GA並列）
- テスト計算

WSLでやってみたところ、**思ったより難しかったです**。

## CUI環境の準備(Windowsの場合)

Windows上でCUI環境を扱うには、とりあえずWindows Subsystem for Linux (WSL)を使うことにしましょう。
これにより、Windows上でLinuxを扱うことができるようになります。
他にもCygwinを使うなどが考えられます。このあたりはお好みに応じて。
Macintoshの場合は特に何もしなくてもできると思いますが、YoshioNishimotoはMacintoshユーザーではないのでよく分かりません。

まずは[https://learn.microsoft.com/ja-jp/windows/wsl/about](https://learn.microsoft.com/ja-jp/windows/wsl/about)おより[WSLをインストール](https://learn.microsoft.com/ja-jp/windows/wsl/install)を参考にWSLをインストールしましょう。
普通にやると、Ubuntu (Linuxディストリビューションの一つ)がインストールされます。
私がやってみたところ、Ubuntu 22.04.3 LTSがインストーされました（2024年4月13日）。

```
自分用メモ
Windowsキー + R → cmd
> wsl --install # cmd command
割と時間がかかる。で、再起動。
Enter new UNIX username: ...
New password: ...
```

なお、WSLを使うにはLinuxのコマンドを勉強する必要があります。
ls, cd, vi(m) (× emacs), cp, mv, rm, mkdirあたりは調べて使えるようにしておきましょう。

## Gitによるソースコードの取得

このチュートリアルはOpenMolcasのチュートリアルなので、Gitに関する説明は最低限です。
より詳しい説明はいろいろなウェブページにあるので、そちらも参照してください。

プログラミングの経験がある方はご存じだと思いますが、Gitは「プログラムのソースコードなどの変更履歴を記録・追跡するための分散型バージョン管理システム」（Wikipediaより）です。
プログラムは往々にしてバグを混入させてしまうものです。
しかも、複数人でプログラムを作成している場合は、適切な管理をしないと誰がどのタイミングでバグを混入させたのか分からなくなってしまいます。
そこで、Gitを用いてプログラムを管理させてやれば、バグが混入したときに簡単（？）にどのタイミングでバグが混入したのかを把握することができますし、過去のある時点へのコードに巻き戻ったりすることもできるというわけです。

このチュートリアルは[GitHub](https://github.co.jp/)上で作成しています。
GitHubというのは、Gitを用いたソフトウェア開発のプラットフォーム（環境・ウェブサイト・アプリと考えれば良い）のことです。
他にも[GitLab](https://about.gitlab.com/ja-jp/)が有名でしょう。
自分のレポジトリを作成するにはアカウントが必要ですが、既存のレポジトリをのぞいたりコードをダウンロードするにはアカウントは必要ありません。
が、LibXCのインストールの段階でおそらくgit cloneをしなければならないため、アカウントを作って公開鍵を登録しておくのが良いかと思います。
`$ ssh-keygen -t rsa`から~/.ssh/id_rsa.pubをGitLabから登録しておきましょう（GitHubユーザーの場合は、git cloneでgithubを使うのが良い？）。

というわけで、ホームディレクトリ上にOpenMolcasのv24.02 (2024年2月時点でのスナップショット。masterブランチでも良い)をインストールすることにしてみます。
[https://gitlab.com/Molcas/OpenMolcas/-/releases](https://gitlab.com/Molcas/OpenMolcas/-/releases)のページから対応するZIPファイル（OpenMolcas-v24.02.zip）でもダウンロードして、
WSLのホームディレクトリ（エクスプローラーからの場所は「\\wsl.localhost\Ubuntu\home\***\」）に配置します

<!--
```
$ cd ~  # ホームディレクトリに移動
$ ls # カレントディレクトリのファイル・ディレクトリを表示
OpenMolcas-v24.02.zip

```

デフォルトだとunzipコマンドが使えないので、管理者ユーザーでapt get installします。

```

# apt install unzip
# exit
$ unzip OpenMolcas-v24.02.zip
...
$ cd OpenMolcas-v24.02
```
ZIPファイルは削除しても良いでしょう。
git cloneを使う場合は次のような感じ。
-->

```
$ cd ~  # ホームディレクトリに移動
$ ls # カレントディレクトリのファイル・ディレクトリを表示
$ git clone https://gitlab.com/Molcas/OpenMolcas.git # リモートのOpenMolcasレポジトリを取得
$ ls
$ cd OpenMolcas
$ ## git checkout v24.02 # v24.02というブランチにきりかえる
```

普通にWSLをインストールするとgitも使えるはずですが、
もしもgitコマンドが見つからない場合はなんとかしてインストールしましょう。

## インストールの準備

もう一つ、WSLはそのままだとコンパイラがインストールされていないようです。
なので、普通にGNUコンパイラなどをインストールしてみます。
余裕があればintel compilerでもインストールしましょう。

```
$ sudo passwd root
[sudo] password for ***:
New password:
Retype new password:
passwd: password updated successfully
$ su
Password:
# apt-get update
# apt install gfortran
# apt-get update
# apt install gfortran
# apt install g++
# apt install cmake
# exit
$
```

「$」は一般ユーザーのプロンプト、「#」は管理者ユーザーのプロンプトです。
<!--
次に、BLASとLAPACKをインストールしましょう。
あとで`./configure-cmake`をするときに失敗するため、
BLASとLAPACKにはいろいろな種類がありますが、intelのmath kernel library (MKL)が使えるようなので、こちらをインストールしてしまいます。

```
$ su
# apt install intel-mkl
# exit
$
```
-->
ここまでがインストール前の準備となります。

## OpenMolcasのコンパイル

まずは一番ベーシックなインストールをしてみましょう。
とりあえず次のコマンドを打ち込んで、どのようなオプションがあるか確認。

```
$ ./configure-cmake
```

できればオプションを検討して欲しい（`--compiler`、`--opt`、`--omp`、`--prefix`あたり）ところですが、まずは何も考えずに

```
$ ./configure-cmake --default
```

すると、おそらくエラーメッセージが出るので対処しましょう（線形代数の計算に必要なBLASとLAPACKというライブラリが見つからないため）。
結局、今回は次のような感じでインストールしました（このようにする必要はない）。

```
$ ./configure-cmake --compiler gnu --prefix /home/nisimoto/OpenMolcas --omp
```

デフォルトでは`--opt dev`になる気がしますが、変なメッセージが出るので`--opt normal`でも良いでしょう。
あとは指示に沿ってコンパイル・インストールしましょう（`make-`のあとはインストールの方法により異なる場合があります）。

```
$./make-gnu_dev_omp
```

「コンパイルを通すのも勉強のうち」との方針なので、エラーが出た場合はなんとかしましょう。

LibXCが上手くいかなかったのでhttps://qiita.com/kkato233/items/1fc71bde5a6d94f1b982 を参照したりしていましたが、
いろいろやっていたのでどこで解決したかよく分かっていないです。
普段の環境だと何も面倒なことは起こらないのですが･･･。

## OpenMolcasのコンパイル（GA並列）

OpenMolcasは数学ライブラリがスレッド並列に対応していれば並列計算が可能（`--omp`オプションが必要）です。
この並列計算は一つの計算機の中での並列計算が可能となるのみで、多くの計算機を使うような並列を行うことはできません（このチュートリアルでそこまでやるかは別として）。
また、スレッド並列は数学ライブラリを使っていない部分の処理は並列されません。

OpenMolcasでスタンダードな並列計算は、Global Arrays (GA)を用いて行います。
GAは並列計算をするためのライブラリで、
これはmessage passage interface (MPI)をラップして分散メモリをアクセスしやすくしているだけで、
実際にはより低レベルのMPIを呼んでいます（私の理解が正しければ）。

このため、まずはMPIでラップをされたコンパイラが必要になります。
apt経由でOpenMPIがインストールできるので、

```
$ su
# apt install libopenmpi-dev
# apt install openmpi-bin
# which mpif90
/usr/bin/mpif90
# which mpicc
/usr/bin/mpicc
# exit
$
```

のような感じでインストール（と確認）ができます。
次にGAをインストールします。

```
$ cd ~
$ git clone git@github.com:GlobalArrays/ga.git
$ cd ga
$ ./autogen.sh
$ ./configure
$ make
$ make check # optional
$ sudo make install # optional
```

あとはGAを使ったOpenMolcasインストール。

```
$ export GA_BUILD=OFF
$ export GAROOT=/usr/local
$ ./configure-cmake --compiler gnu --prefix /home/nisimoto/OpenMolcas --ga /usr/local
$ ./make-gnu_dev_ga
```

以下のテスト計算ではGA並列を使わない`molcas-gnu_dev_omp`で行っていますが、
GA並列を使う場合は`molcas-gnu_dev_ga`を使ってください（インストールができた場合）。

## テスト計算

最後に、テスト計算をしてみましょう。
まずは適当にテスト計算用のディレクトリを作っておきます。

```
$ cd ~
$ mkdir molcas_test
$ cd molcas_test
$ mkdir 00
$ cd 00
```

そして、次のファイルを使って計算をしてみましょう（[00/test.input](input_files/00_test.input)）。

```shell:00_test.input
&GATEWAY
  Coord = 2
  Angstrom
  H 0.0 0.0 0.0
  H 1.0 0.0 0.0
  Basis = 6-31G*
  Group = C1
  NoCD

&SEWARD

&SCF
```

これをコピーして、`cat > test.input`とでもして、入力を待っているところに上のファイルを貼り付けます。
貼り付け方は環境によりますが、私の場合はマウスの中央ボタンを押すと、貼り付けができるようです。
そしてCtrl+Cで`test.input`というファイルができると思われます。

あとは`molcas-gnu_dev_omp`（`molcas-`のあとはインストールの方法により異なります）を使って計算します。
が、そのままでは使えないので、次のような感じにします。

```shell:molcas-gnu_dev_omp
#!/bin/sh
export MOLCAS=/home/nisimoto/OpenMolcas/builds/gnu_dev_omp
${MOLCAS}/pymolcas $*
```

最後の行で、`pymolcas`を使うようにします。
`molcas-gnu_dev_omp`のファイルは必要に応じて名前を変更し、`${HOME}/bin`に置いてパスを通しておくと良いでしょう。

あとは、引数としてインプットファイル（`test.input`）を指定し、
この実行ファイルを実行すると、OpenMolcasを使った計算をすることができます。

```
$ ~/OpenMolcas/molcas-gnu_dev_omp test.input
```

計算自体は`~/OpenMolcas/molcas-gnu_dev_omp test.input`部分だけで結構ですが、このままだと出力結果がターミナルに出て見直すのが難しくなるので、
最後に`> test.out`をつけて、出力結果を`test.out`というファイルに書き出すようにしています。

```
$ ls
test.input
$ ~/OpenMolcas/molcas-gnu_dev_omp test.input > test.out
```

`test.out`の最後に

```
.##################.
.# Happy landing! #.
.##################.
```

と表示されていれば成功です。お疲れ様でした。
GAを使った並列計算をするには、例えば4コアを使う場合は

```
$ ~/OpenMolcas/molcas-gnu_dev_ga -np 4 test.input > test_GA.out
```

のように、`-np 4`みたいなことを書きます（MPIとOpenMPのハイブリッド並列は不明）。
OMP並列・MPI並列ができているかは、`&GATEWAY`のパネル？をチェックします。

```
()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()

                                              &GATEWAY

             launched 4 MPI processes, running in PARALLEL mode (work-sharing enabled)
                       available to each process: 2.0 GB of memory, 1 thread
                                         master pid: 11195
()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()
```

`launched 4 MPI processes`のようなこと（`-np 4`で指定するコア数による）が書いてあれば、GA (MPI)の並列ができています。
OpenMP並列を行った場合は

```
()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()

                                              &GATEWAY

                                   only a single process is used 
                       available to each process: 2.0 GB of memory, 4 threads
                                             pid: 7705 
()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()
```

のような感じで`4 threads`のように表示されます。
もしかするとここに書いてあるとおりにインストールすると、BLAS/LAPACKがOMP並列に対応していないかもしれません。
必要に応じて、マニュアルでBLAS/LAPACKをインストールして、configureおよびコンパイルをし直してみましょう。

想定していたよりインストールが難しかったので、とりあえずここまで。

## 演習問題

- もう少し大きな分子を用いた計算をして、OMP並列とGA並列のパフォーマンスを比較してみましょう。`Coord = 2`は原子数を入れています。`Angstrom`の直後は元素とそれぞれのX軸、Y軸、Z軸方向の座標（単位はAngstrom）を示しています。
- ヒュッケル法でどのような近似をしていたか教えてください。
- 

[^1]: 人名を結ぶのはエンダッシュ（–）が良いとされています。ハイフン（-）ではない。「2013」をF5で変換すると良いでしょう。
[^2]: 計算機の性能を限界まで発揮しようという場合であったり、計算手法に特化したプログラムが必要であったりという場合は一から作成する必要があります。プログラムが汎用的になるほど余計な処理が多くなりますので。
