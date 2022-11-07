<!--
title: asyncioの翻訳memo
tags: クソ記事,初心者
private: false
-->

this article is written at 2022 11 07


### コルーチン
サブルーチンの一般化.
コルーチンは多くのポイントから入ったり出たり,　再会したりすることができる.

#### コルーチン関数
async def がついている関数をコルーチン関数と呼ぶ.
コルーチンを実装するための関数.
コルーチン関数ではawait, async for, async withを使うことができる.

#### コルーチンオブジェクト
コルーチン関数が返す値をコルーチンオブジェクトと呼ぶ.

``` c1.py
import asyncio

async def main():
    print("HELLO")
    await asyncio.sleep(1)
    print("WORLD")

asyncio.run(main())
```
上のコードは”HELLO"の後1秒後, "WORLD"を出力するプログラムである.
コルーチン(関数)mainを実装している.
コルーチン関数なのでawaitを使える.

```c1-1.py
main() #うまくいかない
```
単にコルーチンを呼び出しただけでは動かない.
コルーチンを走らせるには次の3つの方法がある.

- asyncio.run() を使う.
- コルーチンをawaitする.
- コルーチンをTasksとして走らせる asyncio.create_task() を使う.

これら3つを試す前にAwaitable について話す.
Awaitableはコルーチンを走らす3つの方法の全てに関わってくる.

#### Awaitable
コルーチン(オブジェクト), Task, そしてFutureと呼ばれるクラスは Awaitable オブジェクトである.
Awaitable オブジェクトは await 式の中で使うことができる.
asyncioのAPIの多くは, Awaitableオブジェクトを受け取るようにできている.
以下, コルーチンとTaskについて述べる.

##### Awaitableとして振る舞うコルーチン
``` c2.py
import asyncio

async def nested():
    return 42 # 42はコルーチンオブジェクト

async def main():
    nested() # await されないと走らない
    print(await nested()) # Awaitableであるコルーチン
asyncio.run(main())
```
mainの中の最初のnested()はコルーチンとして作られるが, await しないために走らない.
次のnested()はawait するので走り, 42と表示される.
await nested()の返り値はprintで出力される.
[参考url](https://qiita.com/everylittle/items/57da997d9e0507050085)

###### AwaitableとしてのTask
``` c2-2.py
import asyncio

async def nested():
    return 42 # 42はコルーチンオブジェクト

async def main():
    task = asyncio.create_task(nested())
    await task # AwaitableであるTask
asyncio.run(main())
```
Taskはコルーチンを並列に走らせることができる.
この場合, taskが生成された時点でnested()が走り, また, nested()の中断・完了をawait で待つことができる.


コルーチンは基本的な非同期処理の要素である.
基本的にTaskを操ることで非同期処理を実装することになるだろう.

#### Task の作成
asyncio.create(coro)を使って実行をスケジュールし, Taskオブジェクトを返す.
そのTaskオブジェクトはget_running_loop()から返されたループ内で実行される.
現在(のスレッドの中で), 実行中のループがなければエラーRuntimeErrorを出す.

#### Task Group の作成
Task Group とはタスク作成のAPIとグループ内の全てのタスクが終了するまで待機することをうまく組み合わせたもの.
``` c3.py
async def main():
    async with asyncio.TaskGroup() as tg:
        task1 = tg.create_task(some_coro(...))
        task2 = tg.create_task(another_coro(...))
    print("Both tasks have completed now.")
```
async with は全てのtaskが終わるまで待つ.
待ってる間, 新しいtaskがグループに追加される可能性もある.
ただ, 最後のtaskが終了し, async with ブロックが終了したなら新しいtaskはもう追加できない.
