## 目的

ライフゲームの作例を検索してみると、固定長二次元配列を使った作例が多く見られます。しかし二次元配列を用いる場合、

- 二次元配列のサイズをどのくらいにとるのが妥当なのか？
- 二次元配列上はほぼいつも空なのに全画面について計算するのは無駄では？
- 配列の端の存在が全体の計算結果にどんな影響を与えるのか？

など疑問が出てきます。

生きているセルの座標点の集合として取り扱えば、無限の空間に広がり続けるパターンをシミュレーションすることができます。ここでは、Python の set を使ってライフゲームを実装してみます。

## ルールを実装する

set に生きている（「生」状態の）セルの座標が入っていて、set にない座標はすべて「死」であると定義します。この場合、以下の手順でライフゲームの次のステップでの生きているセルを得ることができます。

1. 生きているセルの座標点の集合が与えられます。
2. 生きているセルに隣接する座標すべてのリストを得ます。
3. 上で得たリストに含まれる座標について、その座標について何回リスト上に出現するか個数を数えます。
4. 「次のステップで生きているセルとなる可能性のある座標すべてについて、当該座標に隣接する現時点で生きているセルの個数(隣接生存セル数と呼ぶことにします)」が上記までの手順で得られます。
5. 得られた座標と、隣接生存セル数のペアについて、以下のルールで次ステップの「生」「死」を判定します：
 * 隣接生存セルの数が 3 なら、その座標のセルは元の状態に依らず無条件に「生」です。
 * 隣接生存セルの数が 2 なら、その座標のセルの元の状態が「生」なら「生」、「死」なら「死」です。
 * それ以外の座標については元の状態に依らずすべて「死」です。
6. 判定結果を次ステップの生存セルの座標 set として返します。

このような考えで、リスト内包表記を使って実装したのが以下です。一行が長くなっても構わないならば、一行に収めて書くこともできます：


```python
import collections

def life(dots):
    u'''ライフゲームのルールに従って、与えられた「生」セルの座標 set を元に
    次のステップの「生」セルの座標 set を返す。
    --------
    dots: Set[Tuple[int, int]]
        「生」セルの座標の set

    Returns
    --------
    next_dots: Set[Tuple[int, int]]
        次のステップでの「生」セルの座標の set
    '''
    return {point
            for point, n in collections.Counter([(x + i, y + j)
                                                 for (x, y) in dots
                                                 for i in range(-1, 2)
                                                 for j in range(-1, 2)
                                                 if i != 0 or j != 0]).items()
            if n == 3 or (n == 2 and point in dots)}
```
実際に、これがどのくらい実用的（？）に使えるものなのか気になったので、pygame で作った簡単な画面に出しながら実行できるようにしてみました。
「どんぐり」をファイルで与えると画面いっぱい、いや、画面をはみ出してでもさらに広い領域に広がっていきます。「どんぐり」は無限の彼方に飛んでいくグライダーをいくつも生成しますが、画面サイズに制約されない計算方式なので好きなだけ長時間継続して見ていることができます。

```python
import sys
import life_display
import life_file

def main():
    u'''Life1.05 形式のファイルから初期状態を読み込んで実行する。
    '''
    if len(sys.argv) < 2:
        print('test: <life file>')
        sys.exit(0)
    fps = 200  # フレームレート
    path = sys.argv[1]
    # ファイルから読み込む
    dots = life_file.read_life_105_file(0, 0, path)
    life_file.write_life_105(dots)
    # pygames フロントエンドで表示しながら実行する場合
    # display = life_display.LifeDisplayAndGenerate()
    # pygames フロントエンドで表示しながらアニメーション GIF を作成する場合
    display = life_display.LifeDisplayAndGenerateImages(width=200, height=200)
    for i in range(1, 70):
        display.draw(dots)
        dots = life(dots)
        display.clock_tick(fps)
    display.generate_animation_gif('/tmp/life.gif',
                                   optimize=True, duration=1, loop=0)
if __name__ == '__main__':
    main()
```

![acorn_mini.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/412308/45decc63-fbd2-b970-b235-47f781e78a27.gif)

しょぼい GIF アニメーションですみません。もっと長時間動いている画像をお見せ出来ないのが残念です。

画面表示、ファイルからの初期パターンの読み込みについては別記事で投稿します。

## まとめ

set を使ってスケールフリーなライフゲームを実装しました。
set とリスト内包を使って書いてみると、なんとなく APL 版ライフゲームを思い起こします。
