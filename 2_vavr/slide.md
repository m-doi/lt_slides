今期DDDがいい感じになった
===

---
# 概要

色々工夫してみたらキレイ※なコードになったという話

※個人の感想です

---
# 「ない」ことの表現が改善

## NullObject

例：解約申込

```java
interface Contract {
  EndOrder orderEnd();
}

class ExistContract {
  @Override
  public EndOrder orderEnd() {
  	return new ExistOrder(this.id);
  }
}

class NotExistContract {
  @Override
  public EndOrder orderEnd() {
  	throw new RuntimeException("Not Contracted");
  }
}
```

---

つかうときはこんな感じ

```java
class EndOrderService {
  void orderEnd(User user) {
    Contract cnt = cntRep.find(user);
    EndOrder endOrd = cnt.orderEnd();
    endOrdRep.save(endOrd);
  }
}
```

--- 
# 問題になったこと

- 明らかに存在するコンテキストで必要なメソッドもNOに書かなきゃいけない
- ↑これが嫌でCastを多用し始めた（実行時エラーになる。Javaなのに…）
  ```
  void orderEnd(User user) {
    Contract cnt = cntRep.find(user);
    if(!cnt.isExist()) {
      throw new RuntimeException("Not Contracted");
    }
    EndOrder endOrd = ((ExistContract)cnt).orderEnd();
    endOrdRep.save(endOrd);
  }
  ```

---
## Optionalだと？

```java
void order(User user) {
  Optional<Contract> cntOpt = cntRep.find(user);
  EndOrder endOrd = cntOpt
    .map(Contract::orderEnd)
    .orElseThrow(
    	() -> new RuntimeException("Not Contracted");
    )
  endOrdRep.save(endOrd);
}
```
Castするよりはいいけど、「ない時はエラー」というのはドメイン層に押し込みたい

---
# Eitherつかってみた

「vavr（javaslang）を使ってみよう」ってなったので、Eitherで「ある、なし」を表現してみた
```java
class Contract {
  Either<NotExist, Exist> cont
}
```

- 「ないときXX」「あるときXX」をドメイン層に押し込めた
- NOとそうじゃないやつはそれぞれ必要なメソッドだけ実装すればOK

---
# Validationを使ってみた

- ドメイン層で例外をスローしているとコピペコードっぽいものが増えた
	- 例:「ダメだったらお客様にメールで通知」みたいな要件が出てきた時にboolean型を返すチェック機能が作られたりとか
- ドメイン層で例外スローは禁止にしてValiationにしてみたらキレイになった
```java
class Contract {
  Either<NotExist, Exist> cont

  Validation<Error, EndOrder> orderEnd() {
    return cont.isleft ?
    	Validation.left(Error.create("Not Contracted")) :
    	Validation.right(cont.get().orderEnd());
  }
}
```

---
# XXXPolicyクラス

サービス仕様書（要求仕様書）によくこんな表がでてくる

例：サービスを申込できる条件
（列：ユーザーのステータス、行：申込有無
||正常|利用停止中|解約手続き中|
|:--|:--|:--|:--|
|申込あり|不可|不可|不可|
|申込なし|可能|不可|不可|

---
そのままクラスにしてみた
```java
class ApplyPolicy {

  User usr;
  Order ord;

  Validation<Error, NewOrder> order() {
    // 上記の表に基づいてごにょごにょと
  }
}
```
- テストコードはspockのwhereブロックに前述の表がそのまま出てくる感じ
- サービス仕様書の、あの表に変更があったときにどこを直せばいいかすぐにわかる

---
# まとめ（所感）

- vavrいいね
	- ここではEitherとValidationしか取り上げてないけど、TupleとかOptionとかcollection系も便利
	- Scalaを使うために、まずはチームメンバーに慣れてもらうために使うのはあり
- Policyクラスいいね
	- 仕様書がそのままコードに落ちる感じ
	- 仕様書のクオリティをあげよう（ついては仕様をちゃんと決めよう）というマインドにもつながるんじゃないかな