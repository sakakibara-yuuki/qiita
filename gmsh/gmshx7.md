<!--
title:   gmsh
tags:    クソ記事,初心者
id:      3d58fa9e114bbb726364
private: false
-->


this article is written at 2022 11 21

# 初めに
x7.pyは有限要素法に使える技術がたくさんあるのでちゃんと読むこと


``` python:x7.py
# Meshes are fully described in Gmsh by nodes and elements, both associated to
# model entities. The API can be used to generate and handle other mesh
# entities, i.e. mesh edges and faces, which are not stored by default.

# Let's create a simple model and mesh it:
gmsh.model.occ.addBox(0, 0, 0, 1, 1, 1)
gmsh.model.occ.synchronize()
gmsh.option.setNumber("Mesh.MeshSizeMin", 2.)
gmsh.model.mesh.generate(3)

```

Gmshではノードと要素によってMeshは完全に記述されて, どっちもモデル実体に関連づけれられている.
APIを使用して他のmesh実体(meshの辺と面)を生成し, 扱うことができる.


``` python:x7.py
# Like elements, mesh edges and faces are described by (an ordered list of)
# their nodes. Let us retrieve the edges and the (triangular) faces of all the
# first order tetrahedra in the mesh:
elementType = gmsh.model.mesh.getElementType("tetrahedron", 1)
edgeNodes = gmsh.model.mesh.getElementEdgeNodes(elementType)
faceNodes = gmsh.model.mesh.getElementFaceNodes(elementType, 3)
```

要素のように, meshの辺と面は他のノードによって構成される.
meshの四面体の一次の辺と面を取得してみよう.

<!-- 要素と同様に、メッシュのエッジと面はノード (の順序付けられたリスト) によって記述されます。 -->
<!-- メッシュ内のすべての一次四面体のエッジと(三角形の) 面を取得しましょう. -->

``` python:x7.py
# Edges and faces are returned for each element as a list of nodes corresponding
# to the canonical orientation of the edges and faces for a given element type.

# Gmsh can also identify unique edges and faces (a single edge or face whatever
# the ordering of their nodes) and assign them a unique tag. This identification
# can be done internally by Gmsh (e.g. when generating keys for basis
# functions), or requested explicitly as follows:
gmsh.model.mesh.createEdges()
gmsh.model.mesh.createFaces()

# Edge and face tags can then be retrieved by providing their nodes:
edgeTags, edgeOrientations = gmsh.model.mesh.getEdges(edgeNodes)
faceTags, faceOrientations = gmsh.model.mesh.getFaces(3, faceNodes)
```

各要素のエッジと面は, 特定の要素タイプのエッジと面の正規の方向に対応するノードのリストとして返されます.
Gmshは一意のエッジと面(ノードの順序に関係なく、単一のエッジまたは面)を識別し, それらに一意のタグを割り当てることができます.
この識別は, Gmsh によって内部的に処理することできます(たとえば, 基底関数のキーを生成するとき).
または次のように明示的に要求することができます.

ノードを指定することで, エッジタグとフェイスタグを取得できます。


``` python:x7.py
# Since element edge and face nodes are returned in the same order as the
# elements, one can easily keep track of which element(s) each edge or face is
# connected to:
elementTags, elementNodeTags = gmsh.model.mesh.getElementsByType(elementType)
edges2Elements = {}
faces2Elements = {}
for i in range(len(edgeTags)): # 6 edges per tetrahedron
    if not edgeTags[i] in edges2Elements:
        edges2Elements[edgeTags[i]] = [elementTags[i // 6]]
    else:
        edges2Elements[edgeTags[i]].append(elementTags[i // 6])
for i in range(len(faceTags)): # 4 faces per tetrahedron
    if not faceTags[i] in faces2Elements:
        faces2Elements[faceTags[i]] = [elementTags[i // 4]]
    else:
        faces2Elements[faceTags[i]].append(elementTags[i // 4])
```

要素のエッジと面のノードは要素と同じ順序で返されるため, 各エッジまたは面がどの要素に接続されているかを簡単に追跡できます.


``` python:x7.py
# New unique lower dimensional elements can also be easily created given the
# edge or face nodes. This is especially useful for numerical methods that
# require integrating or interpolating on internal edges or faces (like
# e.g. Discontinuous Galerkin techniques), since creating elements for the
# internal entities will make this additional mesh data readily available (see
# `x6.py'). For example, we can create a new discrete surface...
s = gmsh.model.addDiscreteEntity(2)

# ... and fill it with unique triangles corresponding to the faces of the
# tetrahedra:
maxElementTag = gmsh.model.mesh.getMaxElementTag()
uniqueFaceTags = set()
tagsForTriangles = []
faceNodesForTriangles = []
for i in range(len(faceTags)):
    if faceTags[i] not in uniqueFaceTags:
        uniqueFaceTags.add(faceTags[i])
        tagsForTriangles.append(faceTags[i] + maxElementTag)
        faceNodesForTriangles.append(faceNodes[3 * i])
        faceNodesForTriangles.append(faceNodes[3 * i + 1])
        faceNodesForTriangles.append(faceNodes[3 * i + 2])
elementType2D = gmsh.model.mesh.getElementType("triangle", 1)
gmsh.model.mesh.addElementsByType(s, elementType2D, tagsForTriangles,
                                  faceNodesForTriangles)
```

エッジまたは面ノードを指定すると、新しい独自の低次元要素も簡単に作成できます.
これは, 内部エンティティの要素を作成すると, この追加のメッシュデータがすぐに利用できるようになるため, 内部エッジまたは面で積分または補間を必要とする数値手法(たとえば, 不連続ガラーキン法など)に特に役立ちます(「x6.py」を参照).
たとえば, 新しい離散サーフェスを作成できます...

...四面体の面に対応する一意の三角形で塗りつぶします.


``` python:x7.py
# Since the tags for the triangles have been created based on the face tags,
# the information about neighboring elements can also be readily created,
# useful e.g. in Finite Volume or Discontinuous Galerkin techniques:
for t in tagsForTriangles:
    print("triangle " + str(int(t)) + " is connected to tetrahedra " +
          str(faces2Elements[t - maxElementTag]))

# If all you need is the list of all edges or faces in terms of their nodes, you
# can also directly call:
edgeTags, edgeNodes = gmsh.model.mesh.getAllEdges()
faceTags, faceNodes = gmsh.model.mesh.getAllFaces(3)
```

三角形のタグは面タグに基づいて作成されているため, 隣接する要素に関する情報も簡単に作成できます.
有限体積法または不連続ガラーキン法:

ノードに関するすべてのエッジまたは面のリストだけが必要な場合は, 直接呼び出すこともできます.

``` python:x7.py
# -----------------------------------------------------------------------------
#
#  Gmsh Python extended tutorial 7
#
#  Additional mesh data: internal edges and faces
#
# -----------------------------------------------------------------------------

import sys
import gmsh

gmsh.initialize(sys.argv)

gmsh.model.add("x7")

# Meshes are fully described in Gmsh by nodes and elements, both associated to
# model entities. The API can be used to generate and handle other mesh
# entities, i.e. mesh edges and faces, which are not stored by default.

# Let's create a simple model and mesh it:
gmsh.model.occ.addBox(0, 0, 0, 1, 1, 1)
gmsh.model.occ.synchronize()
gmsh.option.setNumber("Mesh.MeshSizeMin", 2.)
gmsh.model.mesh.generate(3)

# Like elements, mesh edges and faces are described by (an ordered list of)
# their nodes. Let us retrieve the edges and the (triangular) faces of all the
# first order tetrahedra in the mesh:
elementType = gmsh.model.mesh.getElementType("tetrahedron", 1)
edgeNodes = gmsh.model.mesh.getElementEdgeNodes(elementType)
faceNodes = gmsh.model.mesh.getElementFaceNodes(elementType, 3)

# Edges and faces are returned for each element as a list of nodes corresponding
# to the canonical orientation of the edges and faces for a given element type.

# Gmsh can also identify unique edges and faces (a single edge or face whatever
# the ordering of their nodes) and assign them a unique tag. This identification
# can be done internally by Gmsh (e.g. when generating keys for basis
# functions), or requested explicitly as follows:
gmsh.model.mesh.createEdges()
gmsh.model.mesh.createFaces()

# Edge and face tags can then be retrieved by providing their nodes:
edgeTags, edgeOrientations = gmsh.model.mesh.getEdges(edgeNodes)
faceTags, faceOrientations = gmsh.model.mesh.getFaces(3, faceNodes)

# Since element edge and face nodes are returned in the same order as the
# elements, one can easily keep track of which element(s) each edge or face is
# connected to:
elementTags, elementNodeTags = gmsh.model.mesh.getElementsByType(elementType)
edges2Elements = {}
faces2Elements = {}
for i in range(len(edgeTags)): # 6 edges per tetrahedron
    if not edgeTags[i] in edges2Elements:
        edges2Elements[edgeTags[i]] = [elementTags[i // 6]]
    else:
        edges2Elements[edgeTags[i]].append(elementTags[i // 6])
for i in range(len(faceTags)): # 4 faces per tetrahedron
    if not faceTags[i] in faces2Elements:
        faces2Elements[faceTags[i]] = [elementTags[i // 4]]
    else:
        faces2Elements[faceTags[i]].append(elementTags[i // 4])

# New unique lower dimensional elements can also be easily created given the
# edge or face nodes. This is especially useful for numerical methods that
# require integrating or interpolating on internal edges or faces (like
# e.g. Discontinuous Galerkin techniques), since creating elements for the
# internal entities will make this additional mesh data readily available (see
# `x6.py'). For example, we can create a new discrete surface...
s = gmsh.model.addDiscreteEntity(2)

# ... and fill it with unique triangles corresponding to the faces of the
# tetrahedra:
maxElementTag = gmsh.model.mesh.getMaxElementTag()
uniqueFaceTags = set()
tagsForTriangles = []
faceNodesForTriangles = []
for i in range(len(faceTags)):
    if faceTags[i] not in uniqueFaceTags:
        uniqueFaceTags.add(faceTags[i])
        tagsForTriangles.append(faceTags[i] + maxElementTag)
        faceNodesForTriangles.append(faceNodes[3 * i])
        faceNodesForTriangles.append(faceNodes[3 * i + 1])
        faceNodesForTriangles.append(faceNodes[3 * i + 2])
elementType2D = gmsh.model.mesh.getElementType("triangle", 1)
gmsh.model.mesh.addElementsByType(s, elementType2D, tagsForTriangles,
                                  faceNodesForTriangles)

# Since the tags for the triangles have been created based on the face tags,
# the information about neighboring elements can also be readily created,
# useful e.g. in Finite Volume or Discontinuous Galerkin techniques:
for t in tagsForTriangles:
    print("triangle " + str(int(t)) + " is connected to tetrahedra " +
          str(faces2Elements[t - maxElementTag]))

# If all you need is the list of all edges or faces in terms of their nodes, you
# can also directly call:
edgeTags, edgeNodes = gmsh.model.mesh.getAllEdges()
faceTags, faceNodes = gmsh.model.mesh.getAllFaces(3)

# Launch the GUI to see the results:
if '-nopopup' not in sys.argv:
    gmsh.fltk.run()
```