# The Bitcoin Lightning Network

*Scalable Off-Chain Instant Payments*

```
Joseph Poon - joseph@lightning.network
Thaddeus Dryja - rx@awsomnet.org
January 14, 2016
DRAFT Version 0.5.9.2
```

## Abstract

## 1 The Bitcoin Blockchain Scalability Problem

## 2 A Network of Micropayment Channels Can Solve Scalability

...

現状のマイクロペイメントとスケーラビリティの解決策は、第三者である管理業者(Custodian)に自分のコインと他者との残高の更新を信託することで、管理業者にトランザクションをオフロードすることである。第三者に全ての資産を信託することは、取引先リスクと取引コストの面で不利益が生じる。

...


### 2.1 Micropayment Channels Do Not Require Trust

...

もし、チャネル上の残高が [Alice:0.05, Bob:0.05]であり、取引後の残高が[Alice:0.07, Bob:0.03]である場合、ネットワークはどっちの残高が正しいのか知る必要がある。

...

それと同時に、ネットワークに流すとコスト高となり得るため、本当に必要なとき以外はブロックチェーンのタイムスタンプを使わないシステムが望まれる。

その代わり、双方はトランザクションへの署名とこのブロードキャストしないことに合意することができる。
もし双方が、2-of-2マルチシグアドレスに送金するトランザクションを作ったならば、これは現在の残高に対する合意となる。
双方はこの2-of-2トランザクションから自分自身へ0.05返金可能であることに合意したことになる。
この返金トランザクションはブロードキャストしない
ただ、双方とも、それをブロードキャストするのが望ましいと感じた時はいつでもそうできる
ブロードキャストを先延ばしにしている間は、双方は将来この拠出金の残高を変更することに合意しているとみなせる

この残高を更新するには、例えばAlice: 0.07, Bob:0.03など、双方が2-of-2マルチシグアドレスに対する新しい支出を作成すればよい。
にも関わらず、適切に設計しなければ、新しい支出と元々の返金どちらが正しいのか知るよしがないというタイムスタンプ問題が生じる。

とはいえ、タイムスタンプと日付の制限は、Bitcoinのブロックチェーンにある全てのトランザクションの完全な順序づけほど複雑ではない。
マイクロペイメントチャネルでは次の2つの状態〜現在の正しい残高とすでに失効した残高〜だけが必須となる。そこには単一の現在の正しい残高と、おそらく多くの失効した過去の残高がのみ存在することになるだろう。

これは、全ての古いトランザクションを無効にし、新しいトランザクションのみが有効とするスクリプトを考案することにより、可能となる。
無効化は、Bitcoinのアウトプットスクリプトと、他者が双方の拠出額の全てをチャネルの相手方に送付できるような依存トランザクションにより行われる。

...

### 2.2 A Network of chanels

このように、マイクロペイメントチャネルは2者間に関係を作るだけである。全ての人が他の全てとチャネルを作らないといけないのであればスケーラビリティの問題は解決しない。Bitcoinのスケーラビリティ問題はマイクロペイメントチャネルの巨大なネットワークを使うことによって実現できる。

...

## 3 Bidirectional Payment Channels

マイクロペイメントチャネルは、とある取引状態のブロードキャストを先延ばしできるようにする。この契約を執行させるには、一方の当事者に対して数日前（もしくは数日後）にトランザクションをブロードキャストする責任を表現する必要がある。ブロックチェーンは非中央集権型のタイムスタンプシステムであるため、データの妥当性を確定する非中央集権型コンセンサスの構成要素としての時計が使えるうえ、（ブロックチェーン自体の）現在の状態をイベントを順序付ける一つの方法として利用できる。

幾つかの状態がブロードキャストでき、かつ後で無効化できるタイムフレームがあれば、Bitcoinのトランザクションスクリプトによって複雑なコントラクトを作成することができる。すでにハブアンドスポーク型マイクロペイメントチャネル（およびトラステッドペイメントチャネルネットワーク）については先行研究されており、実際にハブアンドスポーク型のネットワークが構築された実例もある。しかしながら、ライトニングネットワークの双方向型マイクロペイメントチャネルは、中間ノードの破綻リスクを軽減して無限に近いスケーラビリティを実現するものであり、付録Aに記載したマリアビリティソフトフォークを必要とする。

複数のマイクロペイメントチャネルの相互接続によって、トランザクション経路のネットワークを作ることが可能になる。各経路がBGPのようなシステムでルーティングできれば、送信者は受信者への特定の経路を選べるようにになるかもしれない。出力スクリプトは受信者によって生成されたハッシュで撹乱される。このハッシュへの入力を明らかにすることにより、受信者の相手方はルートにそって資金を得ることが可能になる。

### 3.1 The Problem of Blame in Channel Creation

前述のペイメントネットワークに参加するためには、当事者がネットワーク上の他の参加者との間にマイクロペイメントチャネルを作る必要がある。
 
#### 3.1.1 Creating an Unsigned Funding Transaction

最初のチャネル出資トランザクションは、このトランザクションの入力へのチャネルの片方または双方の当事者の資金によって作られる。両当事者はこのトランザクションの入力と出力を作るが、これに署名はしない。

...

#### 3.1.2 Spending from an Unsigned Transaction

署名がまだ取り交わされていないトランザクションから支出する必要があるため、ライトニングネットワークは、2-of-2の拠出トランザクションのアウトプットからの支出にSIGHASH_NOINPUTトランザクションを使う。ソフトフォークとして実装されているSIGHASH_NOTINPUTは、全員の署名が完了する前に支出できるようにするもので、トランザクションはそのIDを取得するために新しいsighashフラグなしに署名されている必要がある。

...

#### 3.1.3 Commitment Transactions: Unenforcible Construction

未署名の拠出トランザクションが生成されたあと、双方は初期の確定トランザクションに署名して交換する。これらの確定トランザクションは出資トランザクション（＝親）の2-of-2アウトプットからの支出をもつ。しかしながら、拠出トランザクションのみがブロックチェーン上にブロードキャストされる。

...

互いにいつでも（新しい確定トランザクションが生成されたあとであっても）確定トランザクションをブロードキャストしてよいため、少ない拠出金を受け取る方が、そのアウトプットに両者にとってより多くの価値をもつ確定トランザクションをブロードキャストする強いインセンティブを持つことになる。この結果として、チャネルはすぐさまクローズされ、拠出金が盗難されてしまう。それ故、このモデルの下では、誰もペイメントチャネルを生成できない。

#### 3.1.4 Commitment Transactions: Ascribing Blame

すべての署名された確定トランザクションはブロックチェーン上にブロードキャストしてよいため、どちらかがうまくブロードキャストできた場合には、古い確定トランザクションがブロードキャストされることを防ぐ必要がある。Bitcoinの大量のトランザクションを無効にすることは不可能であり、他の方法が必要となる。

...

しかしながら、この構築方法であっても、単に責任の所在を示すものでしかない。Bitcoinブロックチェーン上の契約を執行することはまだできない。BobはAliceが古いトランザクションをブロードキャストしないことを信じるしかない。
この時点でBobが立証できるのは、半分署名されたトランザクションを根拠に、アリスがそれをやったということのみである。

### 3.2 Creating a Channel with Contract Revocation

### 3.3 Sequence Number Maturity

#### 3.3.1 Timestop

#### 3.3.2 Revocable Commitment Transactions

#### 3.3.3 Redeeming Funds from the Channel: Cooperative Counterparties

#### 3.3.4 Creating a new Commitment Transaction and Revoking Prior Commitments

#### 3.3.5 Process for Creating Revocable Commitment Transactions

### 3.4 Cooperatively Closing Out a Channel

### 3.5 Bidirectional Channel Implications and Summary

## 4 Hashed Timelock Contract (HTLC)

### 4.1 Non-revocable HTLC Construction

### 4.2 Off-chain Revocable HTLC

### 4.3 HTLC Off-chain Termination

### 4.4 HTLC Formation and Closing Order

## 5 Key Storage

## 6 Blockchain Transaction Fees for Bidirectional Channels

## 7 Pay to Contract

## 8 The Bitcoin Lightning Network

### 8.1 Decrementing Timelocks

### 8.2 Payment Amount

### 8.3 Clearing Failure and Rerouting

### 8.4 Payment Routing

### 8.5 Fees

## 9 Risks

## 9.1 Improper Timelocks

## 9.2 Forced Expiration Spam

## 9.3 Coin Theft via Cracking

## 9.4 Data Loss

## 9.5 Forgetting to Broadcast the Transaction in Time

## 9.6 Inability to Make Necessary Soft-Forks

## 9.7 Colluding Miner Attacks

## 10 Block Size Increases and Consensus

## 11 Use Cases

## 12 Conclusion

## 13 Acknowledgements

## Appendix A Resolving Malleability

## References
