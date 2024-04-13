# 00 可視化について

（ほとんどPySCF用ファイルのコピーです）

## 可視化ソフトウェア

分子軌道等、OpenMolcasの計算結果を可視化するためには、可視化ソフトウェアが必要になります。
あるいは、計算したい分子を組み立てるためにも、可視化ソフトウェアを用いる方が便利です。
[かなり多くの種類のソフトウェア](https://en.wikipedia.org/wiki/List_of_molecular_graphics_systems)があるのですが、
ここでは私が使っている・メインで使っていたことがあるソフトウェア（全て英語です）をご紹介いたします。

|ソフトウェア|cubeファイルの可視化|分子モデリング|無償|
|-|-|-|-|
|[MOLDEN](http://cheminf.cmbi.ru.nl/molden/)|〇|×|〇|
|[visual molecular dynamics (VMD)](https://www.ks.uiuc.edu/Research/vmd/)|〇|×|〇|
|[MolView](http://molview.org/)|×|〇|〇|
|[ChemCraft](https://www.chemcraftprog.com/)|〇|〇|△|
|[GaussView](https://gaussian.com/gaussview6/)|〇|〇|×|
|[Avogadro](https://avogadro.cc/)|〇|〇|〇|

とは言え、使いやすいもの・気に入ったものを自分で見つけてください。
私はChemCraft (licensed)とVMDをメインで使っています。
この「量子化学ハンズオン」では、皆さんは主に[Avogadro](https://avogadro.cc/)を用いていくことになると思います。
とりあえずこちらをインストールしておきましょう。
私は使ったことがないので分かりませんが、下記によるとAvogadroを用いて分子モデリング（コンピュータ上で分子を組み立てて、たぶんxyzファイルの生成も可能）も可能なようです。

- [分子モデリングソフト Avogadro を使ってみる](https://yutarine.blogspot.com/2008/11/avogadro.html)

基本的には、有償というだけあってGaussViewが最も高性能です。
多くの計算サーバーには既にインストールされていると思います。
ChemCraftは期限付無償とか、そんな感じだったかと思います。

MOLDENとVMDは、たぶん分子を組み立てることができないです。
MOLDENは軽量なのが特長です。
VMDは割ときれいな構造を書くことができます。

---

ただ、いずれもOpenMolcasの出力ファイル（`*.out`など）を直接可視化することはできません。
計算をしたときに一緒に生成される`*.molden`ファイルを上記可視化ソフトウェアに読み込ませることにより可視化します。
# これは[分子軌道の可視化](05_MO_visualization)で説明します。
