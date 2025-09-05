# ニュースアプリ 分析課題 README v3（インターン配布想定）

> 2日間（合計約8h）で、**意思決定に使える分析レポート**を作る実務体験。正解当てではなく、**筋の通ったロジックで“見える事実”を伝える**ことが目的。

---

## 0. テーマ（今回の焦点）
**W1離脱の特定と、若年女性のカテゴリ導線最適化**  
KPI（**PV / DAU / MAU**）を伸ばすには、**W1リテンション改善**と**初週の行動量・導線の是正**が近道。とくに**10–29歳女性**の初週導線（どのカテゴリから入るか／どれだけ読むか）がボトルネックになっている可能性を、データで確認し改善仮説を出す。

---

## 1. 目的とスコープ
- **ビジネス目的**：ニュースアプリの **PV / DAU / MAU を伸ばす**。
- **今回の課題**：「お客様がニュースアプリをどう使っているか」を把握し、**W1離脱の原因仮説**と**カテゴリ導線の改善案**を導く。
- **アウトプット**（3点）
  1) **分析レポート**（10–15枚）：*結論 → 根拠 → 示唆 → 次アクション*（図表多め／定義・母数明記）
  2) **再現可能ノートブック**（Python/SQL）：データ読込～図表生成をワンクリック再実行
  3) **1ページ要約**（エグゼクティブサマリ）：結論3点＋根拠グラフ＋アクション1–2本
- **非スコープ**：実装／A/B実験の実施、外部データの持ち込み

---

## 2. 学生に配布する情報（前提・定義・背景ストーリー）

### 2.1 データ（CSV/Parquet）
- **CSV**：`users.csv` / `sessions.csv` / `pageviews.csv` / `articles.csv`
- **Parquet**：`users.parquet` / `articles.parquet`、  
  `sessions.parquet`・`pageviews.parquet` はサイズ対策で**2分割版**（`*_part1.parquet`, `*_part2.parquet`）を同梱  
  ※ 生成スクリプトは `PARQUET_MODE = "single" | "split" | "both"` で出力形態を切替可能

### 2.2 期間・規模・イベント
- **期間**：**2025-06-01 ～ 2025-07-27（8週/57日）**
- **規模**：ユーザー約 **2,000–3,000（設定により可変）**
- **カテゴリ**：`{ent, sports, economy, tech, life}`
- **時間帯バイアス**：朝/昼/夕/夜 = **0.40 / 0.25 / 0.20 / 0.15**
- **キャンペーン週**（月曜〜日曜・両端含む）：  
  **2025-06-23 ～ 06-30**, **2025-07-21 ～ 07-27**
  - 当該週は**新規登録者数をブースト**（流入増）
  - **当日PVをブースト**（行動増）
  - **キャンペーン登録者のW1継続率を追加で減衰**（定着は弱め）

### 2.3 KPI 定義（固定）
- **DAU / MAU**：当日／月に **1PV以上**のユーザー数  
- **PV**：日／週／月のページビュー総数  
- **週次リテンション**：`W0=サインアップ週、W1=翌週…`（母数= **W0アクティブ者**、観測可能セルのみ）

### 2.4 背景ストーリー（架空）
> 直近リリースした**アカウント必須のニュースアプリ**。ユーザーの継続利用とPV成長が目標。  
> リリース後、**アカウント作成＋記事閲覧でポイント進呈**するキャンペーンを**2回**（上記週）実施。  
> **新規は増えたが、定着が弱い気配**。特に**若年女性（10–29F）**のW1が課題という声が社内外から出ている。  
> プロダクトは順調にダウンロードされているが、**継続率とPVの底上げ**が当面の焦点。

### 2.5 顧客の声（所有顧客のヒアリング要約）
- 「**朝の通勤時間帯にサクッと読みたい**。長文は後回しになりがち」（20代女性）
- 「**エンタメは開くが、続けて読む記事が見つかりにくい**」（10代女性）
- 「**通知が多い日は開くが、その後は続かない**」（30代男性）
- 「**スポーツの速報は良い**が、**関連記事の回遊が弱い**」（20代男性）
- 「**初回におすすめが刺さらないと戻ってこない**」（20代女性）
- 「**昼休みにまとめて読む**ので、**要約と関連記事の並び**が重要」（30代女性）

---

## 3. 期待する成果（アウトカム）
- **KPIの現状把握**：直近の **PV / DAU / MAU のレベルと推移**（日/週）、イベント週の影響（pre→win→post）
- **主要論点の特定**：
  - (A) 全体の**W1落ちの規模**と**落ちる“場所”（週／セグメント）**
  - (B) **若年女性（10–29F）**の流入→W1到達の**到達率ファネル**とボトルネック
  - (C) **キャンペーンの量と質**（流入増 vs 定着弱）
  - (D) **カテゴリ導線**（初手カテゴリや初週カテゴリMixが継続に与える影響）
- **仮説3本以上**（例）
  - 「**ent/life初手**は**当日PVは伸びるがW1は低い**」
  - 「**tech/economy初手**は**W1が相対的に高い**」
  - 「**W0で3–5PV**の閾値を超えると**W1が跳ねる**」
- **施策案**：各仮説に対し**2–3本**（対象/内容/配置）。**成功指標**と**計測方法**（KPI: PV/DAU/MAU, 補助: W1）を明記

---

## 4. ヒント（探索ガイド）
> 答えではなく**探索の方向**。
1) **KPIの俯瞰**：日次/週次の **PV・DAU・MAU** と、キャンペーン週の **pre→win→post**（中央値）で全体像を見る  
2) **リテンション**：コホート×週で **W1が落ちる場所** を特定（全体→属性）  
3) **初週の“量”**：**W0のPV・セッション長**の分布と**しきい値**（3–5PV付近）を確認  
4) **カテゴリ導線**：**初手カテゴリ／初週Mix × W1** を比較、若年女性で差分を見る  
5) **キャンペーンの質**：**キャンペーン登録者のW1** と **当日PVリフト** を分けて評価（量と質）

---

## 5. 利用データのスキーマと設計意図
**スキーマ**
- `users(user_id, age_group, gender, signup_date, is_campaign_signup)`  
- `sessions(session_id, user_id, session_start, session_end, device)`  
- `pageviews(pv_id, session_id, user_id, article_id, ts)`  
- `articles(article_id, category ∈ {ent,sports,economy,tech,life})`

**設計意図（分析で“見つけてほしい”傾向）**
- **W1は明確に落ちる（全体 W1 ≈ 0.35–0.45）**
- **若年女性は低め（10–19F/20–29F）**
- **キャンペーン週は流入・当日PVが上振れ**する一方、**キャンペーン登録者のW1は弱い**
- **カテゴリ差**：`ent/life` は当日PV寄与が高いがW1は相対的に低く、`economy/tech` はW1が相対的に高い
- **初週行動量の閾値**：**W0で3–5PV** を超えるとW1が跳ね上がる境目がある

---

## 6. 進め方（ステップ）
1) **全体把握**：PV/DAU/MAUの時系列にイベントウィンドウを重ねる  
2) **リテンション特定**：コホート×週でW1落ちの“場所”を特定（全体→属性）  
3) **到達率ファネル**：登録→W0アクティブ→W1（属性×週）を可視化  
4) **カテゴリ導線**：初手カテゴリ／初週カテゴリMix × W1、W0行動量×W1の相関  
5) **イベントの質**：pre→win→postの中央値比較＋**キャンペーン登録者**コホートのW1  
6) **仮説→施策**：優先度（影響×実行容易性）で並べ、**指標・テスト設計**まで落とす  
7) **レポート整形**：結論→根拠→示唆の順で、図表に定義・母数を併記

---

## 7. 参考スニペット（定義確認用・疑似コード）
> そのまま動かすのではなく、**定義**の確認に使用。

```python
# KPI: DAU/MAU/PV
pv_by_day  = pageviews.groupby(pageviews["ts"].dt.floor("D")).size()
dau_by_day = pageviews.groupby([pageviews["ts"].dt.floor("D"), "user_id"]).size().groupby(level=0).nunique()
mau_by_mon = pageviews.groupby([pageviews["ts"].dt.to_period("M").dt.start_time, "user_id"]).size().groupby(level=0).nunique()

# 初手カテゴリ
pv = pageviews.merge(articles[["article_id","category"]], on="article_id", how="left")
first_cat = (pv.sort_values(["user_id","ts"])
               .groupby("user_id").first()["category"].rename("first_category")).reset_index()

# 週次リテンション（W0アクティブ基準）
pv_u = pv.merge(users[["user_id","signup_date","age_group","gender","is_campaign_signup"]], on="user_id", how="left")
pv_u["date"] = pv_u["ts"].dt.floor("D")
pv_u["w_no"] = ((pv_u["date"] - pv_u["signup_date"]).dt.days // 7).astype(int)
wk = pv_u[["user_id","w_no"]].drop_duplicates()

# 観測可能セルだけで算出
last_obs = pageviews["ts"].max().floor("D")
users["signup_week"] = users["signup_date"].dt.to_period("W").dt.start_time
wk = wk.merge(users[["user_id","signup_week","gender","age_group","is_campaign_signup"]], on="user_id", how="left")
active = wk.groupby(["signup_week","w_no","gender","age_group"])['user_id'].nunique().rename("active").reset_index()
w0 = active.query("w_no==0").drop("w_no",axis=1).rename(columns={"active":"w0"})
ret = active.merge(w0, on=["signup_week","gender","age_group"], how="left")
ret["cell_end"] = ret["signup_week"] + pd.to_timedelta(ret["w_no"]*7 + 6, unit="D")
ret = ret[(ret["cell_end"] <= last_obs) & (ret["w0"]>0)]
ret["retention"] = ret["active"]/ret["w0"]

# キャンペーン日判定（pre→win→post比較用）
cam_windows = [(pd.Timestamp("2025-06-23"), pd.Timestamp("2025-06-30")),
               (pd.Timestamp("2025-07-21"), pd.Timestamp("2025-07-27"))]
pv_u["is_campaign_day"] = False
for s,e in cam_windows:
    mask = pv_u["date"].between(s.normalize(), e.normalize(), inclusive="both")
    pv_u.loc[mask, "is_campaign_day"] = True
```
