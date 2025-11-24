### 1. 5R2C係数の定義 ($C, H$)
[cite_start]論文の **Table 2** [cite: 249] [cite_start]の値と **Appendix B** の式 [cite: 1234, 1235] を使用します。

**基本定数（Table 2より）:**
* $A_{win} = 6.0 \, \text{m}^2$
* $A_{wall} = 29.1 \, \text{m}^2$  
* $A_{floor} = 40.0 \, \text{m}^2$
* $Vol = 108.0 \, \text{m}^3$
* $U_{wall} = 1.015 \, \text{W/(m}^2\text{K)}$
* $U_{win} = 2.0 \, \text{W/(m}^2\text{K)}$
* $C_{room} = 3,200,000 \, \text{J/K}$ （これを $C_m$ として使用）

**パラメータ計算式:**
* **$C_m$ (Mass Capacity):**
    * [cite_start]値: `3,200,000` (Table 2 [cite: 249])
* **$C_{air}$ (Air Capacity):**
    * 式: $Vol \times \rho_{air} \times c_{p,air}$
    * [cite_start]※論文に直接値はありませんが、空気ノードの式(Eq. B.11)があるため、物性値（$\rho \approx 1.2 \text{kg/m}^3, c_p \approx 1006 \text{J/kgK}$）から計算してください [cite: 1246]。
* **$H_{em}$ (Ext. Wall):**
    * [cite_start]式: $U_{wall} \times A_{wall}$ [cite: 1234]
* **$H_{w}$ (Window):**
    * [cite_start]式: $U_{win} \times A_{win}$ [cite: 1234]
* **$H_{ms}$ (Mass-Surface):**
    * [cite_start]式: $9.1 \times 2.5 \times A_{floor}$ [cite: 1234]
    * ※論文固有の係数(9.1や2.5)が使われています。
* **$H_{is}$ (Air-Surface):**
    * [cite_start]式: $3.45 \times A_{in}$ [cite: 1235]
    * 単純化するなら $A_{wall} + A_{win} + 2 \times A_{floor}$ で妥当です。
* **$H_{ve}$ (Ventilation/Infiltration):**
    * [cite_start]論文の指示: 機械換気のみの場合はデフォルトで `0.0001` [cite: 1229]。

### 2. 熱取得の分配率 ($\Phi$)
論文の **Eq. (B.7) [cite_start]- (B.9)** [cite: 1236, 1237] に従います。

* **入力:**
    * $\Phi_{int}$: 内部発熱の合計
    * $\Phi_{sol}$: 日射取得の合計

* **分配計算:**
    1.  **空気ノードへ ($\Phi_{ia}$):**
        * [cite_start]$\Phi_{ia} = 0.5 \times \Phi_{int}$ [cite: 1236]
        * （内部発熱の50%は対流として空気に直接入る）
    2.  **表面ノードへ ($\Phi_{st}$):**
        * [cite_start]式: $\Phi_{st} = (1 - \frac{2.5 A_{floor}}{A_{in}} - \frac{H_w}{9.1 A_{in}}) \times (0.5 \Phi_{int} + \Phi_{sol})$ [cite: 1237]
        * （残りの内部発熱と、日射全量のうちの「表面吸収分」）
        * ※注：OCRテキストでは式が2つ並んでいますが、文脈上、係数が掛かっているこちらが表面($T_s$)への分配です。
    3.  **躯体ノードへ ($\Phi_{m}$):**
        * [cite_start]式: $\Phi_{m} = \frac{2.5 A_{floor}}{A_{in}} \times (0.5 \Phi_{int} + \Phi_{sol})$ [cite: 1237]
        * （残りの内部発熱と日射のうち、躯体に吸収される分）

### 3. 初期条件 ($T_{initial}$)
* 論文に明示的な初期値指定はないため、ユーザーの提案通り **$T_m = T_s = T_{air} = \text{initial\_temp}$** で開始する方針でOKです。


