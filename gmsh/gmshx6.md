<!--
title:   gmshのx6.pyのメモ
tags:    Gmsh,初心者
id:      abc9fd708e11fac2a05f
private: false
-->


this article is written at 2022 11 04

```x6.py
# The API provides access to all the elementary building blocks required to
# implement finite-element-type numerical methods. Let's create a simple 2D
# model and mesh it:
gmsh.model.occ.addRectangle(0, 0, 0, 1, 0.1)
gmsh.model.occ.synchronize()
gmsh.model.mesh.setTransfiniteAutomatic()
gmsh.model.mesh.generate(2)

```
APIは要素全てにアクセスすることができ,
有限要素法の数値的な方法を実装するのに必要な確立されたブロックを提供する.
簡単な2Dモデルを作ってメッシュをきる.

```x6.py
# Set the element order and the desired interpolation order:
elementOrder = 1
interpolationOrder = 2
gmsh.model.mesh.setOrder(elementOrder)

def pp(label, v, mult):
    print(" * " + str(len(v) / mult) + " " + label + ": " + str(v))
```
要素の次元と補間の次元を設定する.

```x6.py
# Iterate over all the element types present in the mesh:
elementTypes = gmsh.model.mesh.getElementTypes()
```
メッシュに存在する全ての要素タイプを全て走査する.
違うモデルで実行した時には
[15, 21, 26]
と出たが
15は1-node
21は10-nodeを持つ三角形
10-nodeの内訳として三角形の頂点に3つ, 頂点と頂点の間の辺に2つ(× 3辺)で6つ
三角形の重心に1つの合計10つのnodeである.
26は4-nodeの三次辺
4-nodeの内訳として2点が頂点で頂点と頂点の間の辺に2つ
(つまり21の時の辺)
一つ上にelmentOrderとinterpolationOrderというのが出てきたが,
elementOrderで何次の要素かどうかが決まる.
[gmshの公式9.1 MSH file format](https://gmsh.info/doc/texinfo/gmsh.html)にある.
また, 10-nodeのイメージとして
[10-nodeのイメージ](https://www.researchgate.net/figure/a-Three-noded-triangle-for-first-order-node-based-and-edge-based-basis-functions-b_fig1_228976581)
がある.

```x6.py
# Retrieve properties for the given element type
elementName, dim, order, numNodes, numPrimNodes, localNodeCoord =\
gmsh.model.mesh.getElementProperties(t)
print("\n** " + elementName + " **\n")

```
指定された要素のプロパティを取得する.
ここで出力でLine 2 とか Point 1とか Triangle 3とか出てくるが, これは
2-nodeで構成された辺とか1-nodeで構成された点とか 3-nodeで構成された点とかいう意味

```x6.py
# Retrieve integration points for that element type, enabling exact
# integration of polynomials of order "interpolationOrder". The "Gauss"
# integration family returns the "economical" Gauss points if available, and
# defaults to the "CompositeGauss" (tensor product) rule if not.
localCoords, weights =\
gmsh.model.mesh.getIntegrationPoints(t, "Gauss" + str(interpolationOrder))
pp("integration points to integrate order " +
   str(interpolationOrder) + " polynomials", localCoords, 3)
```
要素タイプの積分点を走査し, 次数"nterpolationOrder"の多項式の正確な積分を可能にする.
可能なら"Gauss"積分族は"economical"なガウス点を返す.
できない場合には, デフォルトでは構造化Gauss(テンソル積)即を返す.

ここで
"integration points to integrate order ~"とあるが,
ここの出力, というかlocalCoordsは座標変換して局所座標にした後の積分点が示される.

どういうことかというと,
Point 1の場合, これは[-1, 1]^3空間(点が(0,0,0)に存在する)の中の(0, 0, 0)点で積分したとなる.
Line 2の場合, [-1, 1]^3空間(直線が(-1,0,0)から(1,0,0)へ引かれている)の中の(0, 0, 0)で積分したとなる.
Triangle 3の場合, [-1, 1]^3空間の(三角形の頂点がそれぞれ(0,0,0), (1,0,0), (0,1,0)に存在する)の(1/3, 1/3, 0)※重心
で積分したということである.

```x6.py
# Return the basis functions evaluated at the integration points. Selecting
# "Lagrange" and "GradLagrange" returns the isoparamtric basis functions and
# their gradient (in the reference space of the given element type). A
# specific interpolation order can be requested using "LagrangeN" and
# "GradLagrangeN" with N = 1, 2, ... Other supported function spaces include
# "H1LegendreN", "GradH1LegendreN", "HcurlLegendreN", "CurlHcurlLegendreN".
numComponents, basisFunctions, numOrientations =\
gmsh.model.mesh.getBasisFunctions(t, localCoords, "Lagrange")
pp("basis functions at integration points", basisFunctions, 1)
numComponents, basisFunctions, numOrientations =\
gmsh.model.mesh.getBasisFunctions(t, localCoords, "GradLagrange")
pp("basis function gradients at integration points", basisFunctions, 3)
```
積分点における基底関数を返す.
LagrangeかGradLagrangeをすれば, アイソパラメトリック基底関数とその勾配を返す.
(要素タイプの空間を参照して)
特定の補間次数をLagrangeNとGradLagrangeN (N = 1, 2, ..)で指定することができる.

N=1も指定できることに注意

その他にサポートされている関数空間として"H1LegendreN"(H1ルジャンドルN), "GradH1LegendreN",
"HcurlLegendreN", "CurlHcurlLegendreN"がある.

```x6.py
# Compute the Jacobians (and their determinants) at the integration points
# for all the elements of the given type in the mesh. Beware that the
# Jacobians are returned "by column": see the API documentation for details.
jacobians, determinants, coords =\
gmsh.model.mesh.getJacobians(t, localCoords)
pp("Jacobian determinants at integration points", determinants, 1)
```
メッシュ内で与えられたタイプの全ての要素のヤコビアン(とその行列式)を積分点で計算することができる.
注意することは, ヤコビアンは列として返されること.
詳しくはAPIドキュメントを見て.

## x6.pyで使った関数
### setOrder(elementOrder) -> None
### getElementTypes() -> elementTypes
### getElementProperties(elementType) -> elementName, dim, order, numNodes, numPrimNodes, localNodeCoord 
### getIntegrationPoints(elementType, "Gauss1") -> localCoords, weights
elementTypeで要素タイプとここでは"Gauss1"ってあるように積分法を与えると数値求積のための情報を得ることができる.
ここで, 積分法は積分族の名前とお望みの次数を繋げたものを使うことができる. (今回だとガウス積分の1次ってこと)
localCoordsはu, v, wの座標でG個の積分点が入る.
つまり, [g1u, g1v, g1w, ..., gGu, gGv, gGw], 見やすくすると[u1, v1, w1, ..., uG, vG, wG] ..?
weightにはq次積分点での重さ[g1q, ..., gGq]が入る.

### getBasisFunctions(elementType, localCoords, "Lagrange") -> numComponents, basisFunctions, numOrientations
numComponents は基底関数の構成要素の数 C を返します. (たとえば, スカラー関数の場合は 1, ベクトル関数の場合は 3)。
basisFunctionsは評価点におけるN基底関数値を返します. 例えば, C=1である場合, gGfN = [g1f1, g1f2, ..., g1fN, g2f1, ...]
G=1でN=1の場合[g1f1u, g1f1v, g1f1w]
C=3の場合, gGfNuvw = [g1f1u, g1f1v, g1f1w, g1f2u, ..., g1fNw, g2f1u, ...]となる.
要素の方向に依存する基底関数の場合, 最初の方向のすべての値が最初に返されます.
numOrientations は, 向きの総数を返します.
wantOrientations が空でない場合は、目的の方向インデックスの値のみを返します。

### getJacobians(elementType, localCoords) -> jacobians, determinants, coords


## 参考資料
以下はかなり参考になりそう
[有限要素法](http://save.sys.t.u-tokyo.ac.jp/~kawai/main_fem/node194.html)
```x6.py
# -----------------------------------------------------------------------------
#
#  Gmsh Python extended tutorial 6
#
#  Additional mesh data: integration points, Jacobians and basis functions
#
# -----------------------------------------------------------------------------

import gmsh
import sys

gmsh.initialize(sys.argv)

gmsh.model.add("x6")

# The API provides access to all the elementary building blocks required to
# implement finite-element-type numerical methods. Let's create a simple 2D
# model and mesh it:
gmsh.model.occ.addRectangle(0, 0, 0, 1, 0.1)
gmsh.model.occ.synchronize()
gmsh.model.mesh.setTransfiniteAutomatic()
gmsh.model.mesh.generate(2)

# Set the element order and the desired interpolation order:
elementOrder = 1
interpolationOrder = 2
gmsh.model.mesh.setOrder(elementOrder)

def pp(label, v, mult):
    print(" * " + str(len(v) / mult) + " " + label + ": " + str(v))

# Iterate over all the element types present in the mesh:
elementTypes = gmsh.model.mesh.getElementTypes()

for t in elementTypes:
    # Retrieve properties for the given element type
    elementName, dim, order, numNodes, numPrimNodes, localNodeCoord =\
    gmsh.model.mesh.getElementProperties(t)
    print("\n** " + elementName + " **\n")

    # Retrieve integration points for that element type, enabling exact
    # integration of polynomials of order "interpolationOrder". The "Gauss"
    # integration family returns the "economical" Gauss points if available, and
    # defaults to the "CompositeGauss" (tensor product) rule if not.
    localCoords, weights =\
    gmsh.model.mesh.getIntegrationPoints(t, "Gauss" + str(interpolationOrder))
    pp("integration points to integrate order " +
       str(interpolationOrder) + " polynomials", localCoords, 3)

    # Return the basis functions evaluated at the integration points. Selecting
    # "Lagrange" and "GradLagrange" returns the isoparamtric basis functions and
    # their gradient (in the reference space of the given element type). A
    # specific interpolation order can be requested using "LagrangeN" and
    # "GradLagrangeN" with N = 1, 2, ... Other supported function spaces include
    # "H1LegendreN", "GradH1LegendreN", "HcurlLegendreN", "CurlHcurlLegendreN".
    numComponents, basisFunctions, numOrientations =\
    gmsh.model.mesh.getBasisFunctions(t, localCoords, "Lagrange")
    pp("basis functions at integration points", basisFunctions, 1)
    numComponents, basisFunctions, numOrientations =\
    gmsh.model.mesh.getBasisFunctions(t, localCoords, "GradLagrange")
    pp("basis function gradients at integration points", basisFunctions, 3)

    # Compute the Jacobians (and their determinants) at the integration points
    # for all the elements of the given type in the mesh. Beware that the
    # Jacobians are returned "by column": see the API documentation for details.
    jacobians, determinants, coords =\
    gmsh.model.mesh.getJacobians(t, localCoords)
    pp("Jacobian determinants at integration points", determinants, 1)

gmsh.finalize()
```
