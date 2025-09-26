## プレゼンテーション資料：『インスピレーション・タロット』
### 1. プロジェクト概要
アプリケーション名: インスピレーション・タロット

コンセプト: 占い師が抱いたインスピレーションと、ランダムに引いたタロットカードの意味をAIが融合させ、世界に一枚だけのオリジナルタロットカードの生成プロンプトを作成するシステム。

目的: 抽象的な「感覚」を具体的な「ビジュアル」に変換する新しい占い体験の技術的検証。

### 2. ペルソナとビジネス価値
ペルソナ: 占い師

課題: JoJoやペルソナなど、タロットカードモチーフの作品のブーム→タロットカードに興味を持つ人が増加傾向にあると思われる。また様々なデザインのタロットカードが世に生まれており、オリジナリティの高いデザインのタロットカードには需要がある。そこで画像生成AIによって常にオリジナルなデザインが出力されるタロット占いソフトは占い師の間で需要がある可能性がある。

ビジネス価値:

顧客にとって、占いの結果がユニークなアートとして視覚化されるため、満足度が向上する。

生成されたカード画像を記念品として提供するなど、新しいサービス展開の可能性がある。

![サンプル1](./sample/generated_f98b4dba-bd1c-4fd4-b503-3900cb41476f2356833320426926422.jpg)

#### プロンプト
```
A stunning and fantastical tarot card illustration, The Star, featuring a graceful glowing jellyfish as a celestial being, pouring light into the deep ocean. The background is a cosmic night sky
```

### 3. システムアーキテクチャ
構成図:

Web 3層構成

Presentation層: (今回は省略したが) Streamlitなど

Application層: Google Colab (Python) - AIの全処理を実行

Data層: tarot_knowledge.txt (RAG用知識ベース), dataset.jsonl (Fine-tuning用データ)

### 4. AIモデルの設計と実装
課題で求められた3つの技術要素（LLM, RAG, Agent）を以下のように実装しました。

LLM (大規模言語モデル)
役割: 「カード情報」と「占い師の印象」という2つの異なる情報を創造的に組み合わせ、高品質な画像生成プロンプト（英語）に変換する役割。

実装: TinyLlama-1.1B-Chat-v1.0 をベースモデルとして採用。独自の学習データ（20例）を用いて、QLoRAによる効率的なファインチューニングを実施した。

RAG (Retrieval-Augmented Generation)
役割: ランダムに引いたカード番号に対応する「意味」と「象徴」の情報を知識ベースから正確に取得（Retrieval）し、LLMへの入力情報として補強（Augment）する。

実装: 意味による類似検索ではなく、ランダムなカード番号をキーとした直接的なデータルックアップとしてRAGの仕組みを応用。tarot_knowledge.txtがその知識データベースとして機能した。

Agent (エージェント)
役割: ユーザーからのリクエストに対し、自律的に複数のステップを実行する司令塔。

実装: Colab Notebook全体が簡易的なAgentとして振る舞う。

Step1: 乱数を生成する。

Step2: 乱数をキーにRAG（知識ベース）からカード情報を取得する。

Step3: 取得した情報と占い師の印象を統合し、ファインチューニング済みLLMに最終的な指示を出す。

Step4: LLMから最終的なプロンプトを受け取り、出力する。

### 5. 結果

結果: 最終的に動作したColab Notebookの実行結果。

入力: 占い師の印象: "どことなく猫っぽい雰囲気の人" + 引いたカード: "8 - 力 (Strength)"

出力:
```
### Instruction:
WARNING: datasets 3.6.0 does not provide the extra 'progress'
✅ ライブラリのインストールが完了しました。

✅ dataset.jsonl を正常に読み込み、整形しました。
データサンプル (整形後):
### Instruction:
      以下の「引いたカードの情報」と「占い師の印象」を創造的に組み合わせて、最高の画像生成プロンプトを作成してください。

      ### Input:
      {"card": "9 - 隠者 (The Hermit)", "impression": "甘いものが好きな人"}

      ### Output:
      a single tarot card, 'The Hermit', a wise old man on a mountain peak, his lantern glows warmly, instead of a star, it contains a delicious floating cupcake, his expression is one of quiet joy, surrounded by a magical candy landscape
Map: 100%
 20/20 [00:00<00:00, 565.87 examples/s]
==((====))==  Unsloth - 2x faster free finetuning | Num GPUs used = 1
   \\   /|    Num examples = 20 | Num Epochs = 30 | Total steps = 90
O^O/ \_/ \    Batch size per device = 4 | Gradient accumulation steps = 2
\        /    Data Parallel GPUs = 1 | Total batch size (4 x 2 x 1) = 8
 "-____-"     Trainable parameters = 50,462,720 of 1,150,511,104 (4.39% trained)

⏳ ファインチューニングを開始します...(10~15分程度かかります)
 [90/90 01:55, Epoch 30/30]
Step	Training Loss
10	1.898500
20	0.925200
30	0.647600
40	0.392000
50	0.177500
60	0.085500
70	0.055200
80	0.046800
90	0.044500
Device set to use cuda:0
✅ ファインチューニングが完了しました！

--- 推論テスト ---
モデルへの入力:
### Instruction:
以下の「引いたカードの情報」と「占い師の印象」を創造的に組み合わせて、最高の画像生成プロンプトを作成してください。

### Drawn Card Info:
- Name: 8 - 力 (Strength)

### Fortune Teller's Impression:
どことなく猫っぽい雰囲気の人

### Output:


モデルの出力:
### Instruction:
以下の「引いたカードの情報」と「占い師の印象」を創造的に組み合わせて、最高の画像生成プロンプトを作成してください。

### Drawn Card Info:
- Name: 8 - 力 (Strength)

### Fortune Teller's Impression:
どことなく猫っぽい雰囲気の人

### Output:
一條たたかったカード、「力」の人が、アーキテクチャや構造のようなものを試みるのが好きな人」を創造的に組み合わ
```
### 6. 結果と考察
成果:

非常に少ないデータ（20例）と短い学習時間にもかかわらず、AIが指示の意図を一部理解し、関連キーワードを含んだ文章を生成しようと試みることを確認できた。

コンセプトの技術的な実現可能性（PoC）を証明できた。

課題と今後の展望:

課題: 生成されるプロンプトの品質がまだ低い。これは圧倒的なデータ不足が原因。

今後の展望:

データセットの拡充: 学習データの質と量を100例以上に増やすことで、出力品質の大幅な向上が見込める。

既存の汎用モデルをAPIで利用：fine-tuningを使うというより、プロンプトを工夫すれば十分の可能性が高い。

画像生成API連携: 実際にStable Diffusionなどの画像生成AIと連携させ、カード画像を生成するまでを自動化する。

タロットアプリを作成：一枚だけでなく、複数枚タロットカードを引くかのように画像を生成するアプリがあるとよいかも。