# 制限活性空間を用いたSCF/PT2計算

以前[多配置SCF計算](06_MCSCF1.md)をしました。
これは、$`\gamma`$は状態（基底状態や励起状態）の波動関数$`\Psi_\gamma`$をスレーター行列式$`\Phi_I`$を用いて
```math
|\Psi_\gamma^\mathrm{MCSCF} \rangle = \sum_I^{N_\mathrm{det}} c_{I,\gamma} |\Phi_I \rangle
```
などと、複数のスレーター行列式の線形結合として波動関数を表現しました。
このとき、どのようなスレーター行列式を考慮するかが問題で、
[多配置SCF計算](06_MCSCF1.md)のときは、
[活性空間（active space）](https://ja.wikipedia.org/wiki/%E5%AE%8C%E5%85%A8%E6%B4%BB%E6%80%A7%E7%A9%BA%E9%96%93)と呼ばれる
分子軌道と電子から構成される「空間」をユーザーが指定して、
この空間でfull CIを行いスレーター行列式を生成するというものでした。
しかし、full CIを行うということは分子軌道と電子の数を気軽に増やすことはできません。
特に点群$`C_1`$での計算だと、12電子12軌道（CAS(12e,12o)）の計算でも、かなり時間がかかったと思います。

そのようなわけで、より大きな活性空間を扱いつつも、スレーター行列式の数を減らす方法が望ましいです。
そこで、制限活性空間（restricted active space; RAS）という方法を用います。
この方法は、まず活性空間をRAS1、RAS2、RAS3という三つの空間に分割します。
そして、RAS2内ではCASと同様にfull CIの計算を行います。
一方、RAS1とRAS3はそれぞれ主に二電子占有とゼロ電子占有される空間となりますが、
励起する・励起される電子の数を制限します。
CAS空間ではすべての分子軌道と電子を用いてfull CIを行いますが、
RAS空間では励起する電子の数を制限した上で、より小さなRAS2空間のみでfull CIを行います。

<!--<img src="https://github.com/YoshioNishimoto/sandbox/blob/figure/MCSCF_1.png" width="1024">-->

一般的には多配置性に寄与しやすいのはフロンティア軌道付近なので、
このあたりの分子軌道と電子をRAS2に含めたいところです。
一方、励起する・されることによりエネルギーが大きく増加しやすい分子軌道に入っている電子はあまり重要でない（はず）なので、
それらをRAS1やRAS3に含めるというのは、理にかなっているのかもしれません。
一見便利に見えますが、（私が認識している限り）次のような問題があります。

- 分子軌道と電子の数が多くなるので活性空間を作るのが面倒
- SCFが収束しづらい
- truncated CIに対応するため、CASSCFとは違いsize-extensiveではなくなる
- 同程度の数のスレーター行列式を用いるなら、CASSCFの方が精度が良く、高速
- 調子の乗るとスレーター行列式の数が多くなるので、結局大して大きな活性空間にできない

しかしながら、RAS1とRAS3の寄与は小さい、けれど活性空間に入れておきたいという場合はありますので、そのような時に便利です。
例えば、金属部分はCASにしたいけれど、リガンド部分はCASに入れるほどではないとか（？）。

なお、RAS2以外すべての二電子占有軌道をRAS1に入れ、すべての非占有軌道をRAS3に入れ、
励起する・される電子の最大数を2にすると、uncontracted MRCISD (multireference configuration interaction singles and doubles)に対応します。

## CASSCF計算

## 活性空間の指定

## 状態平均CASSCF

## 構造最適化について


## 演習問題

- SA2-CASSCF(2e,2o)を用いてethyleneの吸収・蛍光の波長を計算してみましょう。
- [分子軌道の可視化](04_visualization.md)で取り扱った、1,3-butadiene (C<sub>4</sub>H<sub>6</sub>)と1,3,5-hexatriene (C<sub>6</sub>H<sub>8</sub>)でも同様に垂直励起エネルギーの計算を行ってみましょう。CASSCFの計算後に出てくる、natural orbital（とoccupation number）も示してください。
ただし、当然異なる活性空間を定義する必要があるため、きちんと軌道を見ながら定義しましょう。
<!--- CASSCFのsize-extensivity（[単参照電子相関理論](07_SR_corr.md)の演習問題を参照）を検証してみましょう。CASSCFはsize-extensiveな手法です。計算条件は自分で設定してください。ただし、多配置性がほとんどでない系はやめましょう（活性空間の定義も難しくなる）。一般的には、HOMO--LUMOギャップが大きな分子は多配置性が出にくいです。-->
- 2-methylpyrimidineの第一励起はn-π*励起になった記憶があります。
どのように活性空間を設定すると、この状態を正しく記述できそうでしょうか。
活性空間に入れるべき分子軌道（できればCASSCF計算が収束した後のnatural orbital）を示しましょう。
- ethyleneのC–C間距離を100 Angstrom程度に引き延ばしたとき、size-extensiveになるような計算条件を見つけましょう。
  - ethyleneの計算で、どのように活性空間を定義するのが良いでしょうか。
  - CH<sub>2</sub>の計算で、どのように活性区間を定義するのが良いでしょうか。必要に応じてスピン量子数を変更してみましょう（マニュアル参照）。
  - このとき、CASSCF法のsize-extensive（あるいはsize-consistent）になるでしょうか。
- O<sub>3</sub>の基底状態の構造を得てみましょう。実験値は、結合長が1.2717 Angstrom、結合角が116.47°とのことです。[^4]
---

[^1]: 他の研究者から、元に戻すように言われているにもかかわらず戻っていない。自分の手元の普段使いはゼロになるように直しています。`rasscf/neworb_rasscf.f`で`FDIAG(NO1+NT)=FP(II+NFO+NIO+NFI_+ISTFCK)`を`FDIAG(NO1+NT)=0.0d+00`にする。
[^2]: おそらくinactiveとsecondaryはquasi-canonical orbital（inactiveとsecondaryで別個に対角化をするため、それぞれの空間内ではcanonical orbitalですが、全体としてはcanonical orbitalではない）ですが、activeはnatural orbital。
[^3]: J. Stålring, A. Bernhardsson, and R. Lindh, “Analytical gradients of a state average MCSCF state and a state average diagnostic,” [Mol. Phys.](https://doi.org/10.1080/002689700110005642) 99, 103–114 (2001).
[^4]: A. Barbe, C. Secroun, and P. Jouve, “Infrared spectra of <sup>16</sup>O<sub>3</sub> and <sup>18</sup>O<sub>3</sub>: Darling and Dennison resonance and anharmonic potential function of ozone,” 
[J. Mol. Spectrosc.](https://doi.org/10.1016/0022-2852(74)90267-7) 49, 171–182 (1974).
