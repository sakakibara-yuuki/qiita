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


```x6.py
# Retrieve properties for the given element type
elementName, dim, order, numNodes, numPrimNodes, localNodeCoord =\
gmsh.model.mesh.getElementProperties(t)
print("\n** " + elementName + " **\n")

```
指定された要素のプロパティを取得する.

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