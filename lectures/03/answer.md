# 第3回 解答

## 演習

- 第1回（素のLLM）とエージェントの回答の違い：素のLLMは「3レベル」を初級・中級・上級などと一般論から推測して答えることがあり、講座での定義と異なる。エージェントは用語集を検索するため、「基礎（講義内容の確認）」「実装（自分でコードを書く）」「発展（調査や試行）」の3段階だと、講座固有の定義に沿って答えられる。
- `--trace` で見えたツール呼び出し（ツール名と引数）：`LoRAとQLoRAの違いは？` の実行では、用語検索に `search_glossary` が呼ばれる。引数は `query`（調べる用語）で、例えば `[tool call] search_glossary({'query': 'LoRA'})` のように表示される。QLoRAも個別に検索された場合は、同じ形式で `query` にQLoRAが渡る。実際に表示される呼び出しと引数はモデルの実行結果を `--trace` で確認する。
- 1つの質問での LLM 呼び出し回数（`requests`）：固定ではなく、ツール呼び出しの回数などで変わる。実行時の `[usage]` に表示される `requests` の値を記録する（ソースコードだけから実測値は決められない）。
- 追加したツールと、docstring に書いた説明：`make_quiz(lecture: int) -> str` を追加し、`agent.tool_plain(tools.make_quiz)` で登録した。docstring には「指定した回の用語から1つ選び、意味を問う小テストと答えを返す」「学習者が特定の回の用語についてクイズや理解確認を求めたときに使う」と書いた。引数 `lecture` は対象の回（1始まりの整数）。

## 宿題

- 取り組んだレベル：発展（a）
- やったこと：`adk_agent/glossary_agent/agent.py` を作り、演習の `INSTRUCTIONS` と同じ3つのツール（`search_glossary`、`get_lecture_summary`、`make_quiz`）を Google ADK の `Agent` に渡した。モデルは演習と同じ `app.config.MODEL` を使う。`__init__.py` も追加し、リポジトリ直下から `PYTHONPATH=. uv run --with google-adk adk run adk_agent/glossary_agent` で起動できる形にした。
- 結果：両方で「AIE2026の講座の宿題にある『3レベル』とは何ですか？」と質問した。ADK は用語集に沿って「基礎・実装・発展の3段階で、どれか1つで宿題完了」と回答した。Pydantic AI も同じ内容を回答し、`search_glossary({'query': '3レベル'})` を呼び出した。Pydantic AI の `[usage]` は `requests=2`、`tool_calls=1`。ADK の回答はレベルごとの説明に加えて、選び方を促す一文があり、少し詳しかった。
- 気づき・疑問：組み立て方は、Pydantic AI では `Agent(instructions=...)` の後に `agent.tool_plain(...)` を個別に登録するのに対し、ADK では `Agent(instruction=..., tools=[...])` にまとめて渡す。今回は両方とも同じ用語集を参照して正しく回答したが、文章の細部や長さは異なった。ADK の `adk run` の通常表示では、Pydantic AI の `--trace` のようなツール呼び出し一覧や `requests` 数は確認できなかった。
