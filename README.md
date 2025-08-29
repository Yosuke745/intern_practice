# ニュースアプリ 分析課題 README v2（UAT・インターン配布想定）

> 2日間（合計約8h）で、**意思決定に使える分析レポート**を作る実務体験。正解当てではなく、**筋の通ったロジックで“見える事実”を伝える**ことが目的。

---

## 0. テーマ（今回の焦点）
**W1離脱の特定と、若年女性のカテゴリ導線最適化**  
KPI（MAU/DAU/PV）を伸ばすには、**W1リテンション改善**と**ヘビー偏重の緩和**が近道。とくに**10–29歳女性**の初週導線（どのカテゴリから入るか／どれだけ読むか）がボトルネックになっている可能性を、データで確認し改善仮説を出す。

---

## 1. 目的とスコープ
- **ビジネス目的**：ニュースアプリの **MAU / DAU / PV を伸ばす**。
- **今回の課題**：「お客様がニュースアプリをどう使っているか」を把握し、**W1離脱の原因仮説**と**カテゴリ導線の改善案**を導く。
- **アウトプット**（3点）
  1) **分析レポート**（10–15枚）：*結論 → 根拠 → 示唆 → 次アクション*（図表多め／凡例・定義・母数明記）
  2) **再現可能ノートブック**（Python/SQL）：データ読込～図表生成をワンクリック再実行
  3) **1ページ要約**（エグゼクティブサマリ）：結論3点＋根拠グラフ＋アクション1–2本
- **非スコープ**：実装／A/B実験の実施、外部データの持ち込み

---

## 2. 学生に配布する情報（前提・定義）
- **データ**：CSV（`users.csv` / `sessions.csv` / `pageviews.csv` / `articles.csv`）
- **期間**：**2025-06-01 ～ 2025-08-30（13週）**
- **規模**：ユーザー約 **10,000**
- **イベント**：キャンペーン週 **2025-06-21～06-27**、**2025-07-21～07-27**
- **KPI定義（固定）**
  - **DAU/MAU**：当日／月に **1PV以上**のユーザー数
  - **PV**：日／週／月のページビュー総数
  - **週次リテンション**：`W0=サインアップ週、W1=翌週…`（**母数= W0アクティブ者**／**未確定セル除外**）
  - **ヘビー依存**：**Top20%ユーザーのPVシェア**（必要に応じLorenz/Gini）
  - **イベント比較**：**pre（直前7日中央値）→ win（期間中央値）→ post（直後7日中央値）**
- **分析時の注意**：コホート右端切れ（最近登録者のW1/W2未観測）に注意。
- **作図ルール**：本文は日本語、**グラフ内表記は英語**で可。凡例・定義・母数を図表併記。

---

## 3. 期待する成果（アウトカム）
- **主要論点を特定**：
  - (A) 全体の**W1落ちの規模**と**落ちる“場所”（週／セグメント）**
  - (B) **若年女性（10–29F）**の流入→W1到達の**到達率ファネル**とボトルネック
  - (C) **ヘビー偏重度**（Top20%シェア）と、ライト層の初週行動量の浅さ
  - (D) **キャンペーンの量と質**（流入増 vs 定着）
  - (E) **カテゴリ導線**（初手カテゴリや初週カテゴリMixが継続に与える影響）
- **仮説3本以上**：
  - 例）「**ent/life初手**は**当日PVは伸びるがW1は低い**」「**tech/economy初手**は**W1が相対的に高い**」「**W0に3–5PVの閾値**を超えるとW1が急上昇」等。
- **施策案**：各仮説に対し**2–3本**（誰に／何を／どこで）。**成功指標**・**計測方法**（例：DAU、W1、Top20%シェア、new-user PV比）を明記。

---

## 4. ヒント（“肌感”と探索ガイド）
> 答えではなく**探索の方向**。

1) **継続率**：「悪い気がするが、いつ落ちるか不明」→ **コホート×週**のリテンションで“落ちる場所”を特定。  
2) **若年女性**：「構成比が低く増やしたい。どこが詰まる？」→ **到達率ファネル（登録→W0アクティブ→W1）**と**W0行動量×W1**の相関を属性別に。  
3) **ヘビー偏重**：「一部にPVが寄る」→ **Top20%シェア**と**PV/Session分布**。  
4) **ライトの浅さ**：「入ってすぐ出る」→ **W0のPV・セッション長の分布**と**しきい値**探索。  
5) **キャンペーンの質**：「数は増えるが定着は弱い？」→ **pre→win→post**比較と**キャンペーン週コホートのW1**。  
6) **カテゴリ観点**（未確認領域）：**初手カテゴリ／初週カテゴリMix**×**W1**を比較。若年女性の違いに注目。

---

## 5. 利用データと設計意図（サンプルの“仕込み”）
**スキーマ**  
- `users(user_id, age_group, gender, signup_date)`  
- `sessions(session_id, user_id, session_start, session_end, device)`  
- `pageviews(pv_id, session_id, user_id, article_id, ts)`  
- `articles(article_id, category in {ent,sports,economy,tech,life})`

**データで再現できる行動**（＝インサイトに繋がるよう設計）
- **W1は明確に落ちる（全体 W1 ≈ 0.35–0.45）**
- **若年女性は低め（10–19F ≈ 0.15–0.25、20–29F ≈ 0.25–0.35）**
- **Top20%のPVシェアが高い（≈ 0.85–0.90）**
- **キャンペーン週は流入・PVが上振れ（pre比 +20%超）**だが、**W1は弱め**
- **カテゴリの差**：
  - **ent / life 初手**：W0のPVは稼ぐが**W1が相対的に低い**傾向
  - **economy / tech 初手**：W0のPVは中庸でも**W1が相対的に高い**傾向
- **初週行動量の閾値**：**W0で3–5PV**を超えると**W1が跳ね上がる**境目がある
- **セッション長**（`session_end - session_start`）：短すぎるとW1は弱い

> 上記は「検知すべき主要傾向」のレンジとして**受け入れ基準**にも反映。

---

## 6. 進め方（ステップ）
1) **サニティチェック**：欠損、期間、件数、重複／キー整合を確認  
2) **全体把握**：DAU/MAU/PVの時系列＋イベントウィンドウを重ねる  
3) **リテンション特定**：コホート×週でW1落ちの“場所”を特定（全体→属性）  
4) **到達率ファネル**：登録→W0アクティブ→W1（属性×週）を可視化  
5) **ライト/ヘビー**：PV分布、Top20%シェア、Lorenz/Gini（任意）  
6) **カテゴリ導線**：初手カテゴリ／初週カテゴリMix×W1、W0行動量×W1の相関  
7) **イベントの質**：pre→win→postの中央値比較＋キャンペーン週コホートのW1  
8) **仮説→施策**：優先度（影響×実行容易性）で並べ替え、**指標・テスト設計**まで落とす  
9) **レポート整形**：結論→根拠→示唆の順で、図表に凡例・定義・母数を併記

---

## 7. 受け入れ基準（妥当性チェック）
- **定義の厳守**：W1母数= W0アクティブ／未確定セル除外／Top20%算出方法の明記／pre→win→postは同定義
- **主要傾向の検出**：
  - W1落ち（全体 W1 ≈ 0.35–0.45）
  - 属性差（10–19F、20–29Fの低さ）
  - ヘビー偏重（Top20% ≈ 0.85–0.90）
  - キャンペーンの量の上振れと質の弱さ
  - **カテゴリ×W1の差**と**W0行動量の閾値**の示唆
- **デリバラブル品質**：仮説3本＋施策2–3本、各施策に成功条件・計測指標、再現ノートブック

---

## 8. よくある落とし穴
- **W1母数の誤り**／**右端未確定セルを含む**
- **平均だけで判断**（中央値や分位・分布も）
- **“差がある”で止まる**（**どの地点**で差が生じるか到達率で描く）
- **pre→win→postの基準ぶれ**
- **カテゴリを横断で混ぜる**（初手と初週Mixを分けてみる）

---

## 9. 参考スニペット（定義確認用・疑似コード）
> そのまま動かすのではなく、**定義**の確認に使用。

```python
# 初手カテゴリの抽出例（ユーザーごとに最初のPVのカテゴリ）
pv = pageviews.merge(articles[["article_id","category"]], on="article_id", how="left")
pv["date"] = pv["ts"].dt.floor("D")
first_pv = (pv.sort_values(["user_id","ts"]) 
             .groupby("user_id").first()["category"].rename("first_category").reset_index())

# 週次リテンション（W0アクティブ基準）
pv_u = pv.merge(users[["user_id","signup_date"]], on="user_id", how="left")
pv_u["w_no"] = ((pv_u["date"] - pv_u["signup_date"]).dt.days // 7).astype(int)

wk = pv_u[["user_id","w_no"]].drop_duplicates()
users["signup_week"] = users["signup_date"].dt.to_period("W").dt.start_time
wk = wk.merge(users[["user_id","signup_week","gender","age_group"]], on="user_id", how="left")

active = wk.groupby(["signup_week","w_no","gender","age_group"])['user_id'].nunique().rename("active").reset_index()
w0 = active.query("w_no==0").drop("w_no",axis=1).rename(columns={"active":"w0"})
ret = active.merge(w0, on=["signup_week","gender","age_group"], how="left")

last_obs = pageviews["ts"].max().floor("D")
ret["cell_end"] = ret["signup_week"] + pd.to_timedelta(ret["w_no"]*7 + 6, unit="D")
ret = ret[(ret["cell_end"] <= last_obs) & (ret["w0"]>0)]
ret["retention"] = ret["active"]/ret["w0"]

# Top20% PVシェア
pv_per_user = pageviews.groupby("user_id").size().sort_values(ascending=False)
k = max(1, int(len(pv_per_user)*0.20))
top20_share = pv_per_user.iloc[:k].sum() / pv_per_user.sum()

# キャンペーン pre→win→post（中央値）
def median_in(sr, start, end):
    return sr.loc[(sr.index>=start)&(sr.index<=end)].median()

signups_by_day = users.groupby(users["signup_date"].dt.floor("D")).size()
pv_by_day = pageviews.groupby(pageviews["ts"].dt.floor("D")).size()

camp = (pd.Timestamp("2025-07-21"), pd.Timestamp("2025-07-27"))
pre  = (camp[0]-pd.Timedelta(days=7), camp[0]-pd.Timedelta(days=1))
post = (camp[1]+pd.Timedelta(days=1), camp[1]+pd.Timedelta(days=7))

sign_med_pre = median_in(signups_by_day,*pre)
sign_med_win = median_in(signups_by_day,*camp)
sign_lift = (sign_med_win - sign_med_pre)/sign_med_pre
```
