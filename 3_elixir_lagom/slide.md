ElixirとLagom
===

---
# 概要
2017年上期に個人的に興味をもって勉強したことまとめ


---
# actorモデル

## なぜ興味をもったか？

- インフラに何かあった時に自動復旧する仕組みが欲しい
- CWさんのFalcon：[資料](http://sssslide.com/speakerdeck.com/j5ik2o/chatworkfalsexin-metusezingusisutemuwozhi-eruji-shu) 

## Actorモデルとは？

※個人の見解です
- Actor : サーバー（Webサーバーみたいな）
- Supervisor：サーバー監視する仕組み（nagiosとかzabbixみたいな）
- あとここがわかりやすい：[資料](https://speakerdeck.com/tmaedax/akutamoderufalsehua)


---
# Elixir

- actorを調べていくうちにErlang/Elixirにたどり着いて興味をもった
- Elixirは機能が少ないので、２日あれば基本機能は習得できる
- actorモデルの概要を知るのによかった

---
# Lagom

- akkaを勉強しようとしたら見つけた
- 概要はここが詳しい：[資料](https://www.slideshare.net/negokaz/lagom-reactive-microservices-architecture)
- ES/CQRSにも興味があったので触ってみた
- [成果](https://github.com/doilux/study-lagom)