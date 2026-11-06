# 第3回 解答

## 演習

- 第1回（素のLLM）とエージェントの違い：素のLLMはこのアプリの用語集や講義データを直接検索できないが、エージェントは登録されたツールを呼び出して情報を取得できる。
- `--trace` で見えたツール呼び出し（ツール名と引数）：表示されたツールは `search_glossary(query)`、`get_lecture_summary(lecture)`、`make_quiz(lecture)`。
- 1つの質問での LLM 呼び出し回数（`requests`）：`--list-tools` は `TestModel` を使い、APIキーなしで一覧を表示する。
- 追加したツールと、docstring に書いた説明：`make_quiz(lecture: int) -> str`
指定した回の用語から1つを選び、確認問題と答えを返す。
    「第2回の確認問題を出して」のように、復習用の問題を求められたときに使う。


## 宿題

- 取り組んだレベル：基礎
- やったこと：`app/tools.py` の `search_glossary` の docstring を変更し、比較する用語が複数ある場合は `query` にまとめて入れるようにした。
- 比較した質問：`LoRAとQLoRAの違いは？`
- 結果：
  - 変更前：`search_glossary({'query': 'LoRA'})` と `search_glossary({'query': 'QLoRA'})` の2回。`requests=2`、`tool_calls=2`。
  - 変更後：`search_glossary({'query': 'LoRAとQLoRA'})` の1回。`requests=2`、`tool_calls=1`。
  - どちらも用語集を使って違いを説明し、関係する回として第2回を示した。変更後は2語をまとめて検索したため、ツール呼び出しが1回になった。
- 気づき・疑問：ツールの処理自体を変えなくても、docstring の説明を変えることでモデルが選ぶ引数とツール呼び出し回数が変わった。回答の表現には実行ごとの差もある。



$ uv run python -m app.agent --trace "LoRAとQLoRAの違いは？"
[model] gemini-3.5-flash-lite
結論：LoRAはモデルの重みをそのままに低ランク行列だけを学習する手法ですが、QLoRAはさらにベースモデルの重みを量子化（4ビット化など）してメモリを大幅に節約しながらLoRAで学習する手法です。

それぞれの特徴をまとめると以下の通りです。

* **LoRA（Low-Rank Adaptation）**
  * **定義**：事前学習済みの重みを固定したまま、重みの更新分を2つの小さな低ランク行列で近似し、その部分だけを学習するPEFT（Parameter-Efficient Fine-Tuning）手法です。
  * **特徴**：学習するパラメータ数が非常に少なく、学習結果（アダプタ）も軽量に保存できます。本講座の第2回演習で実際に扱います。

* **QLoRA（Quantized LoRA）**
  * **定義**：ベースモデルの重みを4ビットなどに量子化（圧縮）してメモリ消費を極限まで抑えたうえで、LoRAによる学習を行う手法です。
  * **特徴**：より大規模なモデルを少ないGPUメモリでも効率よく動かす（ファインチューニングする）ことができます。本講座の第2回では、発展課題としてモデルを4ビットで読み込んだ際のメモリや出力の違いと比較することになります。

--- trace ---
[tool call]   search_glossary({'query': 'LoRAとQLoRA'})
[tool return] search_glossary -> [{"term": "LoRA", "lecture": 2, "definition": "事前学習済みの重みは固定したまま、重みの更新分を2つの小さな行列の積（低ランク行列）で表し、その小さな行列だけを学習するPEFT手法。学習するパラ…
[usage] RunUsage(cost=Decimal('0.0010674'), details={'text_prompt_tokens': 1058}, input_text_tokens=1058, input_tokens=1058, output_tokens=300, requests=2, tool_calls=1)