# ニュースアプリ 分析課題 README（UAT用・インターン配布想定）

この課題は、**ニュースアプリ** の利用状況データを使い、
意思決定に使える**分析レポート**を2日間（合計約8h）で作る実務体験です。
「正解の分析」を当てることではなく、**筋の通ったロジックで“見える事実”を伝える**ことが目的です。

---

## 1. 目的とスコープ

- **ビジネス目的**：ニュースアプリの **MAU / DAU / PV を伸ばす**。
- **今回の課題**：「お客様がニュースアプリをどう使っているか」をデータで把握し、**改善仮説**を導く。
- **アウトプット**
  - ① **分析レポート**（10～15枚）…*結論 → 根拠 → 示唆 → 次アクション*（図表多め、凡例・定義・母数明記）
  - ② **再現可能なノートブック**（Python/SQL）…データ読み込み～図表生成がワンクリックで再実行可
  - ③ **1ページ要約**（エグゼクティブサマリ）
- **非スコープ**：実装やA/B実験の実施、外部データの持ち込み

---

## 2. 利用データ

- 形式：CSV（`users.csv` / `sessions.csv` / `pageviews.csv` / `articles.csv`）
- 期間：**2025-06-01 ～ 2025-08-30（13週）**
- 規模：ユーザー 約 **10,000**
- データ辞書（最小限）
  - `users`: `user_id`, `age_group`（"10-19","20-29",…）, `gender`（"M"/"F"）, `signup_date`
  - `sessions`: `session_id`, `user_id`, `session_start`, `session_end`, `device`（"Android","iOS","Web"）
  - `pageviews`: `pv_id`, `session_id`, `user_id`, `article_id`, `ts`
  - `articles`: `article_id`, `category`（"ent","sports","economy","tech","life"）
- **イベント（提供）**：キャンペーン期間は **2025-06-21～06-27** と **2025-07-21～07-27**

> ヒント：週単位の分析では**右端切れ**（最近の登録者はW1/W2が未観測）に注意。

---

## 3. KPI 定義（ここだけ固定）

- **DAU / MAU**：当日 / 月に **1PV以上**のユーザー数
- **PV**：日 / 週 / 月のページビュー総数
- **週次リテンション**：`W0 = サインアップ週`、`W1 = 翌週` …  
  - コホートごとに **W0アクティブ者**（その週に1PV以上）を母数に、各週のアクティブ率を算出
  - **観測右端の未確定セルは除外**（例：サインアップから14日未満のユーザーはW1母数に含めない）
- **ヘビー依存**：**Top 20% のユーザーが占める PV シェア**（任意で **Lorenz曲線 / Gini係数** も可）
- **イベント比較軸**：**直前7日中央値 vs 期間中央値（pre → win → post）**  
  ＋参考：**新規ユーザーの当日PV比率**（その日登録ユーザーのPVが全体に占める割合）

---

## 4. お客様の「肌感ヒント」（方向だけ／解き方は自由）

> *以下は「感じていること」。答えを示すものではありません。*

1) **継続率**  
- ひとこと：**「継続率が悪い気はするが、いつ落ちるのかは分からない」**  
- 見るもの：**週ごとのリテンションカーブ**（全体 → コホート → セグメント）

2) **若年女性の拡大**  
- ひとこと：**「若い女性はユーザー構成上少ない。増やしたいが、どこがボトルネックか分からない」**  
  （属性×継続のデータは未確認）  
- 見るもの：登録→W1→W2の**到達率（属性×週）**、**初週行動量と継続の関係**（若年女性に限らず比較）

3) **ヘビー偏重**  
- ひとこと：**「PVが一部の人に寄っている気がする」**  
- 見るもの：**Top20% PVシェア**、（任意）**Lorenz/Gini**、**PV/セッション分布**

4) **ライト層の浅さ**  
- ひとこと：**「ライトは入ってもすぐ出る感じ」**  
- 見るもの：**W0のPV/セッション・セッション長の分布**、**W0行動量とW1継続の相関**

5) **キャンペーンの“量と質”**  
- ひとこと：**「キャンペーン中は数は増えるが、定着は弱いかも」**  
- 見るもの：**pre→win→post** のサインアップ / PV（中央値）、**新規当日PV比**、**キャンペーン週コホートのW1**

> 注：**カテゴリ観点**はお客様は「見たことがない」設定（＝課題感は未形成）。必要と思えば自由に深掘ってOK。

---

## 5. 進め方（ステップガイド）

1. **全体把握**：基本KPI（DAU/MAU/PV）と登録時系列、イベントウィンドウ重ね
2. **継続の場所を特定**：週次リテンション（全体→コホート→属性）。**どの週で落ちる？**
3. **偏りを見る**：PV分布（Top20%・Lorenz）。ライト/ヘビーの行動量を見比べる
4. **ボトルネック仮説**：若年女性の流入～W1到達までの**どこで落ちるか**を可視化（到達率ファネル）
5. **イベントの質検証**：pre→win→postの比較と、**キャンペーン週コホートのW1**
6. **（任意）自由探索**：必要と感じれば**カテゴリなど**の差分（Δ）や相関も確認

---

## 6. 受け入れ基準（合否ではなく妥当性確認）

- **定義が正しい**
  - 週次リテンションは **W0アクティブ者**を母数・**未確定セル除外**
  - Top20% の算出方法が明記されている（ユーザー単位でPVを並べ、上位20%のPV合計 / 全体PV）
  - イベントは **pre（直前7日中央値）→ win（期間中央値）→ post** の**同じ軸**で比較
- **検知すべき主要傾向（レンジは例）**
  - リテンション：**W1の顕著な落ち**（全体 W1 ≈ **0.35–0.45**）
  - セグメント差：例）**10–19F ≈ 0.15–0.25**, **20–29F ≈ 0.25–0.35**
  - ヘビー依存：Top20% PV シェア **0.85–0.90**
  - キャンペーン：サインアップ/日次PVの期間中央値が **preより上振れ**（例：**+20%超**）。**新規当日PV比**も上昇
- **デリバラブル品質**
  - 仮説は**3本以上**、**施策案**と**計測指標**（成功条件・想定影響度）まで
  - レポートは**意思決定者向けの順序**（結論→根拠→詳細）。図表は**凡例・定義・母数**を明記
  - ノートブックは**再実行可能**（パス、乱数種、依存ライブラリを固定）

---

## 7. よくある落とし穴

- **W1母数の誤り**（W0アクティブ者で割る／未確定セルを含めない）  
- **平均だけで判断**（中央値や分位・分布を見る）  
- **“差がある”で終わる**（**どの地点**で差が生まれるかを到達率で描く）  
- **イベント比較の基準ぶれ**（pre→win→postを**同じ定義で**）

---

## 8. 形式・提出物

- **レポート**：PDF/HTML（図表は埋め込み、凡例・定義・母数明記）
- **ノートブック**：`.ipynb`（データ読込～図表生成まで）
- **1ページ要約**：PDF（結論3点・根拠グラフ・次アクション1～2本）

---

## 9. 参考スニペット（Python擬似コード）

> そのまま動かすのではなく、**定義の確認**に使ってください。

```python
# 週次リテンション（W0アクティブ基準）
pv_u = pageviews.merge(users[["user_id","signup_date"]], on="user_id", how="left")
pv_u["date"] = pv_u["ts"].dt.floor("D")
pv_u["w_no"] = ((pv_u["date"] - pv_u["signup_date"]).dt.days // 7).astype(int)

wk = pv_u[["user_id","w_no"]].drop_duplicates()
users["signup_week"] = users["signup_date"].dt.to_period("W").dt.start_time
wk = wk.merge(users[["user_id","signup_week"]], on="user_id", how="left")

active_counts = wk.groupby(["signup_week","w_no"])["user_id"].nunique().rename("active").reset_index()
w0 = active_counts.query("w_no == 0")[["signup_week","active"]].rename(columns={"active":"w0"})
ret = active_counts.merge(w0, on="signup_week", how="left")

last_obs = pageviews["ts"].max().floor("D")
ret["cohort_start"] = ret["signup_week"]
ret["cell_end"] = ret["cohort_start"] + pd.to_timedelta(ret["w_no"]*7 + 6, unit="D")
ret = ret[(ret["cell_end"] <= last_obs) & (ret["w0"] > 0)]
ret["retention"] = ret["active"] / ret["w0"]

# Top20% PVシェア
pv_per_user = pageviews.groupby("user_id").size().sort_values(ascending=False)
k = max(1, int(len(pv_per_user)*0.20))
top20_share = pv_per_user.iloc[:k].sum() / pv_per_user.sum()

# イベント比較（pre→win→post）
def median_in_range(sr, start, end):
    return sr.loc[(sr.index >= start) & (sr.index <= end)].median()

daily_signups = users.groupby(users["signup_date"].dt.floor("D")).size()
daily_pv = pageviews.groupby(pageviews["ts"].dt.floor("D")).size()

camp = (pd.Timestamp("2025-07-21"), pd.Timestamp("2025-07-27"))
pre = (camp[0] - pd.Timedelta(days=7), camp[0] - pd.Timedelta(days=1))
post= (camp[1] + pd.Timedelta(days=1), camp[1] + pd.Timedelta(days=7))

sign_med_pre  = median_in_range(daily_signups, *pre)
sign_med_win  = median_in_range(daily_signups, *camp)
sign_lift     = (sign_med_win - sign_med_pre) / sign_med_pre
```

# 10
- Google Colab / Jupyter（Python 3.10+）
- 必要ライブラリ：pandas, numpy, matplotlib
