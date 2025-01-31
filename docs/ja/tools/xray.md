# AWS X-Ray

## サンプリングルール

X-Ray を使用したサンプリングルールは、AWS コンソール、ローカル設定ファイル、またはその両方で構成できます。ローコンフィグレーションはコンソールで設定されたものをオーバーライドします。

!!! success
	可能な限り X-Ray コンソール、API、CloudFormation を使用してください。これにより、アプリケーションのサンプリング動作を実行時に変更できます。

次の各基準についてサンプルレートを個別に設定できます。

* サービス名 (例: billing、payments)
* サービスタイプ (例: EC2、コンテナ)  
* HTTP メソッド
* URL パス
* リソース ARN
* ホスト (例: www.example.com)

ベストプラクティスは、問題の診断とパフォーマンスプロファイルの理解に十分なデータを収集する一方で、管理できないほど多くのデータを収集しないようなサンプルレートを設定することです。たとえば、ランディングページへのトラフィックの 1% をサンプリングしながら、支払いページへのリクエストの 10% をサンプリングすることは、強力な可観測性の慣行とうまく整合します。

100% のトランザクションをキャプチャしたい場合があるかもしれません。ただし、トレースはワークロードへのアクセスの法的監査を目的としたものではないことに注意してください。 

!!! warning
	トレースは[監査またはフォレンジック分析](../../signals/traces/#trace-data-is-not-intended-for-forensics-and-auditing)を目的としたものではないため、100% のサンプルレートを避けてください。これは、X-Ray (デフォルトで UDP エミッタを使用) がトランザクショントレースを決して失うことがないという誤った期待を生み出す可能性があります。

ルールとして、トランザクショントレースのキャプチャーは、スタッフへの過度の負荷や AWS の請求額の増加を招くべきではありません。ワークロードが発生させるデータ量を学習しながら、トレースを環境にゆっくりと追加してください。

!!! info 
	デフォルトでは、X-Ray SDK は 1 秒ごとに最初のリクエストと、それ以降のリクエストの 5% を記録します。  

!!! success
	許容できるリザーバサイズを必ず設定してください。リザーバサイズは、キャプチャするリクエストの最大数/秒を決定します。これにより、悪意のある攻撃、望ましくない料金、構成エラーから保護されます。

## デーモンの設定

X-Ray デーモンは、分析のためにテレメトリを X-Ray データプレーンに送信する作業をオフロードすることを目的としています。
そのため、ソースアプリケーションが実行されているサーバー、コンテナ、インスタンスで余分なリソースを消費しないようにする必要があります。

!!! success
	ベストプラクティスは、X-Ray デーモンを別のインスタンスやコンテナで実行し、[関心の分離](../../faq/#what-is-the-separation-of-concerns) を強制し、ソースシステムが邪魔されないようにすることです。

!!! success
	Kubernetes のようなコンテナオーケストレーションパターンでは、サイドカーとして X-Ray デーモンを運用するのが一般的なプラクティスです。

デーモンには安全なデフォルト設定があり、EC2、ECS、EKS、Fargate の環境で、ほとんどの場合、追加の設定なしで動作できます。
ただし、ハイブリッドやその他のクラウド環境の場合は、リモート環境を統合するためにダイレクトコネクトや VPN を使用している場合は、[VPC エンドポイント](https://docs.aws.amazon.com/vpc/latest/privatelink/concepts.html) を反映するように `Endpoint` を調整することができます。

!!! tip
	ソースアプリケーションと同じインスタンスや仮想マシンで X-Ray デーモンを実行する必要がある場合は、`TotalBufferSizeMB` 設定を調整して、X-Ray が許容できる以上のシステムリソースを消費しないようにすることを検討してください。

## アノテーション

AWS X-Ray は、トレースとともに任意のメタデータを送信することをサポートしています。これらは*アノテーション*と呼ばれます。アノテーションは、トレースを論理的にグループ化できる強力な機能です。インデックスも作成されるため、単一のエンティティに関連するトレースを簡単に検索できます。

X-Ray の自動インスツルメンテーション SDK を使用する場合、アノテーションが自動的に表示されないことがあります。コードにアノテーションを追加する必要があり、これによりトレースが大幅に充実し、X-Ray Insights の生成、アノテーションに基づくメトリクス、システム動作からのアラームとアノマリ検出モデルの作成、ユーザーに影響を与えるコンポーネントが観測されたときのチケットの自動化と修復の方法が作成されます。 

!!! success
	アノテーションを使用して、環境内のデータフローを理解してください。

!!! success
	アノテーションされたトレースのパフォーマンスと結果に基づいてアラームを作成します。
