# 多参照摂動理論

多参照摂動理論（multi(-)reference perturbation theory; MRPT）の計算をしてみましょう。
どのような計算かというと、MP2と同様に二電子励起の電子配置を摂動的に考慮する事で、電子相関エネルギーを計算します。
ただし、MP2はHartree–Fock法、すなわち一つのスレーター行列式で記述した波動関数を参照して摂動理論を適用した計算を行いました（単参照摂動理論）。
一方MRPTでは、MCSCF法（実際の実装としてはCASSCF法であることが多い）に対して摂動理論を適用した計算を行います。
おそらく、複数のスレーター行列式（または配置状態関数）を参照して摂動計算を行うため、多参照摂動理論と呼ばれています。
CASSCFの段階で静的電子相関を取り入れ、摂動理論により動的電子相関を取り入れるというのが方針で、
どちらの電子相関もバランス良く取り入れられるため、精度の高い計算ができると言われています。
[単参照電子相関理論](05_correlation.md)で出てきた配置間相互作用や結合クラスターを多配置波動関数に応用することもできます（MRCIやMRCCと略される）が、計算コストは非常に高いです。

詳細までは説明しませんが、MRPTには様々な種類があります。
摂動論は基本的にゼロ次ハミルトニアンの定義は任意（もちろん、計算のしやすさを考慮したり、摂動が小さくなるように選びますが）なので、
ゼロ次ハミルトニアンの定義の方法により、様々なMRPTが定義できます。
一番有名なのは、CASPT2 (complete active space second-order perturbation theory)という手法でしょう。
他にもNEVPT2 (*N*-electron valence state)やMCQDPT2 (multi-configuration quasi-degenerate)という手法もよく知られています。
ちなみに、これらのMRPTで一つのスレーター行列式に対しての計算を行うと、MP2の結果に一致します。
そのような方法のうち、OpenMolcasではCASPT2を扱うことができます。
CASPT2に関しては、OpenMolcasこそがde facto standardと言っても良いでしょう。
単状態はもちろんのこと、数種類の多状態の計算や、さまざまなシフト、そして（一応）構造最適化を行うことができます。

エネルギーとしては、まず一次の波動関数への補正$`\Psi^{(1)}`$を次のような感じで定義しておきます。
```math
\Psi^{(1)} = \sum_{pqrs} T_{pqrs}|\Phi_{pqrs} \rangle
```
$`pqrs`$で二電子励起を表し、$`T_{pqrs}`$が励起強度（どの電子配置が波動関数にどの程度寄与するかを表す）、
$`\Phi_{pqrs}`$は参照波動関数から二電子励起をした電子配置となります。
すると、一次の波動関数$`\Psi^{1}`$は、CASSCFで得るゼロ次波動関数$`\Psi^{(0)}`$との和、
すなわち
```math
\Psi^{1} = \Psi^{(0)} + \Psi^{(1)}
```
により表すこととなります。
このとき、エネルギーはHylleraas（汎）関数
```math
E^\mathrm{PT2} = 2 \langle \Psi^{(1)} | \hat{H} | \Psi^{(0)} \rangle
+ \langle \Psi^{(1)} | \hat{H}_0 - E_0 | \Psi^{(1)} \rangle
```
の形で表されますから、一次の波動関数への補正は
```math
\frac{\partial E^\mathrm{PT2}}{\partial T_{pqrs}} = 2 \left ( \langle \Phi_{pqrs} | \hat{H} | \Psi^{(0)} \rangle
+ \langle \Phi_{pqrs} | \hat{H}_0 - E_0 | \Psi^{(1)} \rangle \right ) = 0
```
となるように、変分的に$`T_{pqrs}`$を決めて波動関数およびエネルギーを計算するようにしてやります。
呪文みたいですね。
$`T_{pqrs}`$を決めるには$`(\partial E^\mathrm{PT2})/(\partial T_{pqrs}) = 0`$という方程式を解く必要があるわけですが、
MP2の場合と違い第二項が簡単にできない[^1]ため、繰り返し計算を行う必要があります。
しかも、この繰り返し計算は数値的に不安定な方程式をなる場合が多い（intruder state problem）ため、real shiftまたはimaginary shiftを用いる必要があります。
具体的には、第二項の値が小さくなりすぎる際に数値的に不安定となるため、第二項の値がゼロにならないようにシフト（適当な値を足す）をしてやるということです。
このため、一般的には次のような方程式を解くこととなります。
```math
\frac{\partial E^{\prime\mathrm{PT2}}}{\partial T_{pqrs}} = 2 \left ( \langle \Phi_{pqrs} | \hat{H} | \Psi^{(0)} \rangle
+ \langle \Phi_{pqrs} | \hat{H}_0 - E_0 + E_\mathrm{shift}| \Psi^{(1)} \rangle \right ) = 0
```
$`E_\mathrm{shift}`$がシフトを表す項です。
real shiftは古い手法で、シフトの値が結果へ大きく影響する場合があります。
現代的には、よりスマートな方法であるimaginary shiftを用いるのが良いでしょう（結果への影響が小さい）。
実際のエネルギーとしては、シフトを含まない式（上の方でHylleraas汎関数と書いた式）で計算します。
$`T_{pqrs}`$はシフトを含めた式で計算するけれど、エネルギーはシフトを含まない式を用いるという雰囲気です。
こうすることで、シフトによる影響を少なくしています。
なお、IPEAシフトはまた別物です。

なお、外部のプラグイン（？; QCMaquis）を使うことでDMRGをゼロ次波動関数として使うNEVPT2の計算もできるようですが、私はインストールが上手くいった試しがありません。

## CASPT2の計算

とりあえず、いつも通りethyleneを使って計算をしてみましょう。
間違っていなければ[07_ethylene_SS.input](input_files/07_ethylene_SS.input)と同一です。
```
&GATEWAY
  Coord
  6
  ethylene
  C1              -0.658467        0.000000        0.000000
  C2               0.658467        0.000000        0.000000
  H3              -1.225694        0.914334        0.000000
  H4              -1.225694       -0.914334        0.000000
  H5               1.225694        0.914334        0.000000
  H6               1.225694       -0.914334        0.000000
  Basis = 6-31G*
  Group = C1
  RICD

&SEWARD

&SCF

&RASSCF
  Inactive = 7 
  RAS2   = 2 
  nActEl = 2 0 0 

&CASPT2
  IPEA = 0.00
  IMAGinary = 0.20
```
`&CASPT2`というモジュールで、CASPT2の計算を行います。
普通に単状態のCASPT2計算をするだけであれば、あまりオプションは必要ありません（実際、なにもなくても計算はできる）が、

- `IPEA`: ionization potential–electron affinityシフト。CASPT2は励起エネルギーを過小評価することが多いので、それを補正するようなシフトです。いろいろ面倒なのでゼロにしておきます。
- `IMAGinary`: imaginaryシフト。0.20にした意味は特にないので、好きな値を使って結構です。基本的には小さい値ほど結果への影響が少ないため、小さめにしましょう。

ぐらいは必要かなと思います。

CASPT2計算の詳細としては、
```
++    CASPT2 specifications:
      ----------------------

      Type of calculation                        SS-CASPT2
      Fock operator                              state-specific
      IPEA shift                                 0.00
      Real shift                                 0.00
      Imaginary shift                            0.20
      The input orbitals will be transformed to quasi-canonical
--
```
は一応見ておきましょう。
- `Type of calculation`: SS-CASPT2 (SS; single-state or state-specific)は、CASSCFで得たそれぞれの状態に対して摂動計算を行う（だけ）ということを意味します。そうでない計算はMS-CASPT2のセクション参照。
- `Fock operator`: ゼロ次ハミルトニアンを定義するときに用いるFock演算子の詳細です（細かいので省略）。
- `IPEA shift`: 前述
- `Real shift`と`Imaginary shift`: 前述。real shiftは使わなくても良いでしょう。

「quasi-canonical」というのは、二電子占有軌道空間（inactive）、活性空間（active）、仮想軌道空間（secondary）それぞれでFock行列を対角化することにより得る軌道のことです。
CASPT2は活性空間の軌道をどのように選んでも構わない（エネルギーが変わらない）ため、どのような軌道を用いても原理的には構わないのですが、
活性空間の軌道をFock行列を対角化できるように選ぶことで計算コストを下げられる部分があるため、OpenMolcasでは(quaci-)canonical分子軌道を使っています。

そして、それから次のような繰り返し計算を行います。
```
 The contributions to the second order correlation energy in atomic units.
-----------------------------------------------------------------------------------------------------------------------------
  IT.      VJTU        VJTI        ATVX        AIVX        VJAI        BVAT        BJAT        BJAI        TOTAL       RNORM  
-----------------------------------------------------------------------------------------------------------------------------
   1    -0.000000   -0.001199    0.000000   -0.029490   -0.006632   -0.005071   -0.044363   -0.140080   -0.226836    0.002501
   2    -0.000000   -0.001224    0.000000   -0.029684   -0.006654   -0.005075   -0.044376   -0.140085   -0.227099    0.000212
   3    -0.000000   -0.001225    0.000000   -0.029685   -0.006654   -0.005075   -0.044378   -0.140085   -0.227103    0.000026
   4    -0.000000   -0.001225    0.000000   -0.029684   -0.006654   -0.005075   -0.044378   -0.140086   -0.227101    0.000003
-----------------------------------------------------------------------------------------------------------------------------
```
`VJTU`や`VJTI`は、摂動的な二電子励起の方法を表しています。
MP2の場合は占有軌道空間と仮想軌道空間の二つしかないため、二電子励起の方法は二つとも占有軌道空間から仮想軌道空間へ励起するという一通りしかありません。
一方、CASPT2の場合は二電子占有軌道空間・活性空間・仮想軌道空間の三つがあるため、二電子励起の方法（組み合わせ）がたくさん存在します。
たとえば、`VJTU`は二電子占有軌道空間(J)から活性空間(V)と、活性空間内（U → T）の励起の組み合わせを意味します。
`VJTI`は二電子占有軌道空間(J, I)から活性空間(V, T)への二電子励起を意味します。
そのようなわけで、8種類の組み合わせができ、トータルとしては8つの項の和である`TOTAL`と示されている部分が、
摂動的な相関エネルギー（大まかにはCASPT2計算により得たいエネルギー）に対応します。
`RNORM`はresidual norm（残差ノルム）のことで、繰り返し計算の収束判定に用いられます。

結果としては
```
  FINAL CASPT2 RESULT:

      Reference energy:         -78.0595401636
      E2 (Non-variational):      -0.2271010975
      Shift correction:          -0.0011076646
      E2 (Variational):          -0.2282087621
      Total energy:             -78.2877489257
      Residual norm:              0.0000003189
      Reference weight:           0.93091

      Contributions to the CASPT2 correlation energy
      Active & Virtual Only:         -0.0050752983
      One Inactive Excited:          -0.0740614288
      Two Inactive Excited:          -0.1479643704
```
となります。最終的なCASPT2エネルギーとしては、`Total energy:             -78.2877489257`（単位はatomic unit）のところを用います。
これは`Reference energy`（CASSCFのエネルギー）と`E2 (Variational)`（摂動により得たエネルギー）の和です。
`E2 (Non-variational)`というのは、繰り返し計算により得られる摂動エネルギーのことです。
すなわち、シフトを含んだエネルギー（定義はしていませんが、上の式で言う$`E^{\prime\mathrm{PT2}}`$）です。
一方、`E2 (Variational)`というのが、シフトの項を含まない摂動エネルギー（$`E^\mathrm{PT2}`$）です[^2]。
我々が「CASPT2による（摂動）エネルギー」というのは、この`E2 (Variational)`を用いて計算した値（ここで言う`Total energy:             -78.2877489257`）を採用します。

## MS-CASPT2の計算

MS(multistate)-CASPT2は、状態平均のCASSCF (SA-CASSCF)を参照してCASPT2の計算を行います。
通常のCASPT2では、(SA-)CASSCFで得た状態それぞれに対して摂動計算を行います。
このとき一次の波動関数は、摂動計算の際に参照する波動関数から二電子励起を行うことにより得ますから、
摂動計算の間は状態平均に用いた他の状態を無視することとなります。
しかし、実際には一次の波動関数は直交しているべきです。
また、CASSCFで状態間の相互作用が強い（特に縮重領域）場合に、単一の参照波動関数で摂動をうまく記述できるかも分かりません。
そこで、摂動的に得た一次の波動関数を用いて有効ハミルトニアン
```math
H_{\alpha\beta}^\mathrm{eff} = \langle \Psi_\alpha^{(0)} | \hat{H} | \Psi_\beta^{(0)} \rangle
+ \frac{1}{2} \left (
\langle \Psi_\alpha^{(0)} | \hat{H} | \Psi_\beta^{(1)} \rangle +
\langle \Psi_\beta^{(0)} | \hat{H} | \Psi_\alpha^{(1)} \rangle \right )
```
を作ってやり、このハミルトニアンを対角化することによりエネルギー（および一次の波動関数）を計算しようというのがMS-CASPT2です。
なお、SA-CASSCFを用いてCASPT2を行うことがMS-CASPT2と言うのではありません。
有効ハミルトニアンの対角化までを含めた計算を行うことがMS-CASPT2と言います。

MS-CASPT2にはいくつかの変種があります。
とりあえずは通常のMS-CASPT2か、XMS-CASPT2 (extended multistate)を使うようにしましょう。
計算上の違いは

- Fock行列が単状態(MS-CASPT2)または状態平均(XMS-CASPT2)
- 参照空間（確かgeneralized Fockをゼロ次波動関数で挟んだもの）の非対角項を無視(MS-CASPT2)するか、対角化をして直交させる(XMS-CASPT2)

となっています（詳細はたいへん難しい）。
MS-CASPT2の方が精度がよい（？）と言われる場合があったりもしますが、ポテンシャルエネルギー面が交わる領域（円錐交差）での記述があまりよろしくないです。
XMS-CASPT2はこのポテンシャルエネルギー面の問題を解決できる手法です。
このため、円錐交差領域の計算をする場合は、XMS-CASPT2が望ましいです。
それ以外の場合はケースバイケースです。

インプットファイルとしては、次のような感じにします。
[07_ethylene_MS.input](input_files/07_ethylene_MS.input)も参照。
```
&CASPT2
  IPEA = 0.00
  IMAGinary = 0.20
  MULT = ALL
```
`&RASSCF`の段階で`CIROOT`が適切に設定され、SA-CASSCFができていなければいけません。
`MULT = ALL`で、SA-CASSCFのエネルギー計算に含められたすべての状態を用いたMS-CASPT2計算を行います。
`ALL`ではなく露わにMS-CASPT2に用いる状態を指定することも可能ですが、あまりやらないと思います。
一方、次のように（`XMULT`）するとXMS-CASPT2計算となります。
```
&CASPT2
  IPEA = 0.00
  IMAGinary = 0.20
  XMULT = ALL
```

ethyleneのSA3-CASSCF(2e,2o)の計算からMS-CASPT2を行った際のアウトプットとしては、次のような感じになりました。
```
  Total CASPT2 energies:
::    CASPT2 Root  1     Total energy:    -78.28577154
::    CASPT2 Root  2     Total energy:    -77.96775036
::    CASPT2 Root  3     Total energy:    -77.75630745

++ Multi-State CASPT2 section:
********************************************************************************
  MULTI-STATE CASPT2 SECTION
--------------------------------------------------------------------------------


  Effective Hamiltonian matrix (Symmetric):

                1               2               3
   1       -78.28577154
   2        -0.00000000    -77.96775036
   3        -0.02657369      0.00000000    -77.75630745

       Total MS-CASPT2 energies:
::    MS-CASPT2 Root  1     Total energy:    -78.28710193
::    MS-CASPT2 Root  2     Total energy:    -77.96775036
::    MS-CASPT2 Root  3     Total energy:    -77.75497707

       Eigenvectors:
            0.99874915     -0.00000000     -0.05000137
            0.00000000      1.00000000      0.00000000
            0.05000137     -0.00000000      0.99874915

--
```
`Total CASPT2 energies`というのは、それぞれの状態ごとにCASPT2計算を行っただけの値です（すなわち上でやったSS-CASPT2）。
そして、`Effective Hamiltonian matrix`が有効ハミルトニアン（$`H_{\alpha\beta}^\mathrm{eff}`$）を示しています。
対角要素がSS-CASPT2の値で、非対角要素が遷移密度行列を計算するなどして頑張って計算します。
この有効ハミルトニアンを対角化すると、固有値として`Total MS-CASPT2 energies`が得られ、
固有ベクトルが`Eigenvectors`として得られます。
最終的なMS-CASPT2のエネルギーは、`MS-CASPT2 Root  X     Total energy`のところからとってくることとなります。

## 構造最適化について

やや時間がかかりますが、CASPT2を用いた構造最適化が可能です。
例は[07_ethylene_opt.input](input_files/07_ethylene_opt.input)参照。
基本的には他の手法と同様に、`&SLAPAF`を追加し、`> do while`と`> end do`みたいなEMILコマンドで囲むだけでできるようになっていると思います。
例では`&MCLR`と`&ALASKA`が露わに書いてありますが、これらは省略可能です。

## 演習問題

計算手法の詳細はお任せします。

- 分子軌道をinactive, active, secondaryの三つに分けて二電子励起を考えると、9種類（または10種類）の励起の組み合わせができそうですが、
CASを参照としたCASPT2では上記の通り8つの励起クラスのみ考えます。
考えていない1つの励起クラスはどのようなものでしょう。
そして、なぜ考えなくても良いのでしょうか。
- CASSCFやCASPT2を用いて、基底状態N<sub>2</sub>のポテンシャルエネルギーカーブを計算してみましょう。
[単参照電子相関理論](05_correlation.md)のところでも計算したと思いますので、特に原子間距離が長い領域に着目して単参照電子相関理論と結果を比較してみましょう。
- 2-methylpyrimidineのn-π*垂直励起エネルギーは、CASSCFとCASPT2の結果を比較すると何が言えるでしょうか。
実験値は3.78 eV（L. Alvarez-Valtierra et al. J. Phys. Chem. A 111, 12802 (2007).)らしいですが、これは実は0-0遷移エネルギーなので垂直励起エネルギーとは少し違います。
必要があれば、IPEAシフトを0.25にした計算を試してみましょう。
- Cytosineのπ-π\*垂直励起エネルギーは実験的に4.6 eVあたりであることが知られています（M. P. Fülscher et al. J. Am. Chem. Soc. 117, 2089-2095 (1995).のTable 4）。
できるだけ再現してみてください。計算方法は指定しませんが、π-π\*励起になっているかは確認しましょう。
- （うまい計算条件が分からないので保留）LiFのS<sub>0</sub>とS<sub>1</sub>のポテンシャルエネルギーカーブ（3から7 Angstromまで）を計算してみましょう。
対称性はナシ、基底関数は6-31G(d)、活性空間はCAS(6e,6o)で、SCFは三状態の平均をとるようにしましょう。
SS-CASPT2とXMS-CASPT2で比較をして、どちらがそれっぽいか考えましょう。
この分子の自由度を踏まえると、二つの状態のポテンシャルエネルギーカーブはどのような振る舞いをするのが自然でしょうか。

---

[^1]: MP2の場合は、特にcanonical分子軌道を用いる場合に第二項を$`pqrs`$を基底として表したときに対角行列（要素は軌道エネルギーの和と差で表せる）となるため、逆行列が簡単に計算できる。
[^2]: VaritionalとNon-variationalの意味がよく分からないというのが正直なところですが、どうやら式の形に起因している模様。実際にexcitation amplitudeが(non-)variationalであるかは関係ないらしい。
