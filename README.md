# Voyager: 大規模言語モデルを用いたオープンエンドな具現化エージェント
<div align="center">

[[Webサイト]](https://voyager.minedojo.org/)
[[arXiv]](https://arxiv.org/abs/2305.16291)
[[PDF]](https://voyager.minedojo.org/assets/documents/voyager.pdf)
[[ツイート]](https://twitter.com/DrJimFan/status/1662115266933972993?s=20)

[![Python Version](https://img.shields.io/badge/Python-3.10-blue.svg)](https://github.com/MineDojo/Voyager)
[![GitHub license](https://img.shields.io/github/license/MineDojo/Voyager)](https://github.com/MineDojo/Voyager/blob/main/LICENSE)
______________________________________________________________________


https://github.com/MineDojo/Voyager/assets/25460983/ce29f45b-43a5-4399-8fd8-5dd105fd64f2

![](images/pull.png)


</div>

私たちは、Minecraft の世界を継続的に探索し、多様なスキルを獲得し、人間の介入なしに新たな発見を行う、LLM を搭載した最初の具現化生涯学習エージェントである Voyager を紹介します。Voyager は次の 3 つの主要コンポーネントで構成されています: 1) 探索を最大化する自動カリキュラム、2) 複雑な行動を保存・取得するための実行可能なコードで構成された、成長し続けるスキルライブラリ、3) プログラム改善のために環境フィードバック、実行エラー、自己検証を組み込んだ、新しい反復的プロンプティング機構。Voyager はブラックボックス方式のクエリで GPT-4 と対話するため、モデルパラメータのファインチューニングは不要です。Voyager によって開発されたスキルは時間的に拡張可能で、解釈可能かつ合成的であり、エージェントの能力を急速に強化し、破滅的忘却を軽減します。実験的に、Voyager は強力な文脈内生涯学習能力を示し、Minecraft をプレイする際に卓越した熟練度を発揮します。これまでの SOTA と比較して、3.3 倍多くのユニークなアイテムを取得し、2.3 倍長い距離を移動し、主要なテックツリーのマイルストーンを最大 15.3 倍速くアンロックします。Voyager は学習したスキルライブラリを新しい Minecraft ワールドで活用し、ゼロから新しいタスクを解決できますが、他の手法は一般化に苦戦します。

このリポジトリでは、Voyager のコードを提供しています。このコードベースは [MIT ライセンス](LICENSE) のもとで公開されています。

# インストール
Voyager には Python ≥ 3.10 と Node.js ≥ 22 が必要です。Ubuntu 20.04、Windows 11、macOS でテスト済みです。以下の手順に従って Voyager をインストールしてください。

## Python のインストール
```
git clone https://github.com/MineDojo/Voyager
cd Voyager
pip install -e .
```

## Node.js のインストール
Python の依存関係に加えて、以下の Node.js パッケージをインストールする必要があります:
```
cd voyager/env/mineflayer
npm install -g npx
npm install
cd mineflayer-collectblock
npx tsc
cd ..
npm install
```

## Minecraft インスタンスのインストール

Voyager は Minecraft ゲームに依存しています。Minecraft ゲームをインストールし、Minecraft インスタンスをセットアップする必要があります。

[Minecraft ログインチュートリアル](installation/minecraft_instance_install.md) の手順に従って、Minecraft インスタンスをセットアップしてください。

## Fabric Mods のインストール

Voyager のすべての機能をサポートするには、fabric mods をインストールする必要があります。すべての mod で正しい Fabric バージョンを使用するようにしてください。

[Fabric Mods のインストール](installation/fabric_mods_install.md) の手順に従って mod をインストールしてください。

# はじめに
Voyager は言語モデルとして OpenAI の GPT-4o 系モデル (既定: 各エージェントは gpt-4o / gpt-4o-mini) を使用します。Voyager を使用するには OpenAI API キーが必要です。[こちら](https://platform.openai.com/account/api-keys) から取得できます。

インストールが完了したら、次のように Voyager を実行できます:
```python
from voyager import Voyager

# mc_port を使用することもできますが、azure_login を強く推奨します
azure_login = {
    "client_id": "YOUR_CLIENT_ID",
    "redirect_url": "https://127.0.0.1/auth-response",
    "secret_value": "[OPTIONAL] YOUR_SECRET_VALUE",
    "version": "fabric-loader-0.14.18-1.19", # Voyager がテストされているバージョン
}
openai_api_key = "YOUR_API_KEY"

voyager = Voyager(
    azure_login=azure_login,
    openai_api_key=openai_api_key,
)

# 生涯学習を開始
voyager.learn()
```

* `Azure Login` で初めて実行する場合、コマンドラインの指示に従って設定ファイルを生成するよう求められます。
* `Azure Login` では、自分でワールドを選択し、ワールドを LAN に公開する必要もあります。`voyager.learn()` を実行するとすぐにゲームが起動するので、以下の操作が必要です:
  1. `シングルプレイヤー` を選択し、`新規ワールドを作成` を押します。
  2. ゲームモードを `クリエイティブ`、難易度を `ピースフル` に設定します。
  3. ワールドが作成されたら、`Esc` キーを押して `LAN に公開` を押します。
  4. `チートを許可: ON` を選択し、`LAN ワールドを開始` を押します。まもなくボットがワールドに参加するのが確認できます。

# 学習中のチェックポイントからの再開

学習プロセスを停止し、後でチェックポイントから再開したい場合は、次のように Voyager をインスタンス化します:
```python
from voyager import Voyager

voyager = Voyager(
    azure_login=azure_login,
    openai_api_key=openai_api_key,
    ckpt_dir="YOUR_CKPT_DIR",
    resume=True,
)
```

# 学習済みスキルライブラリを使って特定のタスクで Voyager を実行する

学習済みスキルライブラリを使って特定のタスクで Voyager を実行したい場合は、まずスキルライブラリのディレクトリを Voyager に渡します:
```python
from voyager import Voyager

# 最初に skill_library_dir を指定して Voyager をインスタンス化します。
voyager = Voyager(
    azure_login=azure_login,
    openai_api_key=openai_api_key,
    skill_library_dir="./skill_library/trial1", # 学習済みスキルライブラリを読み込む
    ckpt_dir="YOUR_CKPT_DIR", # 新しいディレクトリを自由に使用してください。新しいイベントは ckpt_dir に記録されるため、スキルライブラリと同じディレクトリは使用しないでください。
    resume=False, # これは学習ではないため、スキルライブラリから再開しないでください。
)
```
次に、タスク分解を実行できます。注意: タスク分解が論理的でない場合があります。出力されたサブゴールに問題があることに気づいたら、分解を再実行できます。
```python
# タスク分解を実行
task = "YOUR TASK" # 例: "Craft a diamond pickaxe"
sub_goals = voyager.decompose_task(task=task)
```
最後に、学習済みスキルライブラリを使ってサブゴールを実行できます:
```python
voyager.inference(sub_goals=sub_goals)
```

有効なスキルライブラリの一覧については、[学習済みスキルライブラリ](skill_library/README.md) を参照してください。

# FAQ
ご質問がある場合は、Issue を開く前にまず [FAQ](FAQ.md) をご確認ください。

# 論文と引用

私たちの研究成果が役に立った場合は、ぜひ引用してください!

```bibtex
@article{wang2023voyager,
  title   = {Voyager: An Open-Ended Embodied Agent with Large Language Models},
  author  = {Guanzhi Wang and Yuqi Xie and Yunfan Jiang and Ajay Mandlekar and Chaowei Xiao and Yuke Zhu and Linxi Fan and Anima Anandkumar},
  year    = {2023},
  journal = {arXiv preprint arXiv: Arxiv-2305.16291}
}
```

免責事項: このプロジェクトは厳密に研究目的であり、NVIDIA の公式製品ではありません。
