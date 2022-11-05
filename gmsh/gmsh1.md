<!--
title:   gmshのt1.pyのメモ
tags:    クソ記事,初心者
id:      119a8a98b856dce7c87f
private: false
-->


this article is written at 2022 011 4

gmshのチュートリアルの説明をこれからしていく.
ほとんどが翻訳となる予定.

t1.pyから見ていくことにする.

```t1.py
# The Python API is entirely defined in the `gmsh.py' module (which contains the
# full documentation of all the functions in the API):
import gmsh
import sys

# Before using any functions in the Python API, Gmsh must be initialized:
gmsh.initialize()

```
Python APIはgmsh.pyで全て定義されていて, APIで使われる全ての関数についてのドキュメントを含んでいる.
Python APIを使う前に必ずGmshを初期化しなければならない.

ちなみに,

```t1.py
# This should be called when you are done using the Gmsh Python API:
gmsh.finalize()
```
Python APIを使い終わったら閉じなけらばならない

```t1.py
# Next we add a new model named "t1" (if gmsh.model.add() is not called a new
# unnamed model will be created on the fly, if necessary):
gmsh.model.add("t1")

```
次にモデルを追加する.
今回はt1と名前をつけるが, gmsh.model.add()を引数なしで呼ぶと無名のモデルが出来上がる.

```t1.py
# The Python API provides direct access to each supported geometry (CAD)
# kernel. The built-in kernel is used in this first tutorial: the corresponding
# API functions have the `gmsh.model.geo' prefix.

```
Python APIは互いにサポートされてるgeometry kernlに直接アクセスできる.
内蔵kernelは最初のチュートリアルで触る.
特に関係のあるAPI関数の頭はgmsh.model.geoとなっている.

ちなみに,

```t1.py
# Note that starting with Gmsh 3.0, models can be built using other geometry
# kernels than the default "built-in" kernel. To use the OpenCASCADE CAD kernel
# instead of the built-in kernel, you should use the functions with the
# `gmsh.model.occ' prefix.
#
# Different CAD kernels have different features. With OpenCASCADE, instead of
# defining the surface by successively defining 4 points, 4 curves and 1 curve
# loop, one can define the rectangular surface directly with
#
# gmsh.model.occ.addRectangle(.2, 0, 0, .1, .3)
#
# After synchronization with the Gmsh model with
#
# gmsh.model.occ.synchronize()
#
# the underlying curves and points could be accessed with
# gmsh.model.getBoundary().
#
# See e.g. `t16.py', `t18.py', `t19.py' or `t20.py' for complete examples based
# on OpenCASCADE, and `examples/api' for more.
```
Gmsh3.0を立ち上げると, モデルはデフォルトの内蔵kernelではなく, 他のgeometry kernelを使用することに注意する.
OpenCASCADE CAD kernlを内蔵kerelの代わりに使いたい時にはgmsh.model.occ を頭につけなければならない.

異なるCAD kernlは異なる特徴を持つ.
OpenCASCADEでは一連の4点, 4つの曲線, 1つの閉曲線(向き)を決めることよりも,
長方形の表面を指定することで定義される.
gmshモデルと同期した後に, 引いた曲線や点はgmsh.model.getBoundary()で参照できる.
詳しくは t16.py t18.py t19.py t20.pyをみるべし.
さらにはexamples/apiをみるべし.


```t1.py
# The first type of `elementary entity' in Gmsh is a `Point'. To create a point
# with the built-in CAD kernel, the Python API function is
# gmsh.model.geo.addPoint():
# - the first 3 arguments are the point coordinates (x, y, z)
# - the next (optional) argument is the target mesh size close to the point
# - the last (optional) argument is the point tag (a stricly positive integer
#   that uniquely identifies the point)
lc = 1e-2
gmsh.model.geo.addPoint(0, 0, 0, lc, 1)
```

Gmshの要素実体の最初の型は`Point`.
内蔵kernelでpointを作るにはgmsh.model.geo.addPoint()を呼ぶ.
gemsh.model.geo.addPoint()の最初の3つの引数は座標(x,y,z)を表す.
次の(optional)引数はpoint近くの目標メッシュサイズを指定する.
最後の(optional)引数はpointのtagを表す. これは正整数でありpointごとに一意でなければならない.

```t1.py
# Note that in addition to the default ``camelCase'' function name `addPoint',
# the Python API also defines a ``snake case'' alias, i.e. `add_point'. You can
# use either interchangeably; all the tutorials are written using the camelCase
# convention for consistency.

# The distribution of the mesh element sizes will be obtained by interpolation
# of these mesh sizes throughout the geometry. Another method to specify mesh
# sizes is to use general mesh size Fields (see `t10.py'). A particular case is
# the use of a background mesh (see `t7.py').
#
# If no target mesh size of provided, a default uniform coarse size will be used
# for the model, based on the overall model size.
#
# We can then define some additional points. All points should have different
# tags:
gmsh.model.geo.addPoint(.1, 0, 0, lc, 2)
gmsh.model.geo.addPoint(.1, .3, 0, lc, 3)

```
注意しなければならないことはcamelCaseかsnake caseかだ.
Python APIはsnake caseもcamelCaseを定義されているがチュートリアルでは(c++との?)一貫性のためcamelCaseを採用している.

メッシュ要素(以下, ただの要素という)のサイズの分布はgeometry全体を通したメッシュサイズの補間によって決まる.
その他でmeshサイズを指定する方法はt10.pyを見てくれ.
部分的に極める方法はt7.pyを見てくれ

目標メッシュサイズが指定されないなら, デフォルトで全体のモデルサイズに基づいて均一の粗いサイズが決められる.

さて, pointを追加しよう. どのpointも異なるtagを持つ必要がある.

```x6.py
# If the tag is not provided explicitly, a new tag is automatically created, and
# returned by the function:
p4 = gmsh.model.geo.addPoint(0, .3, 0, lc)
```
もし, タグが明示的に与えられないなら, 新しいタグが自動的に作られ, 関数の帰り値として返される.

```x6.py
# Curves are Gmsh's second type of elementery entities, and, amongst curves,
# straight lines are the simplest. The API to create straight line segments with
# the built-in kernel follows the same conventions: the first 2 arguments are
# point tags (the start and end points of the line), and the last (optional one)
# is the line tag.
#
# In the commands below, for example, the line 1 starts at point 1 and ends at
# point 2.
#
# Note that curve tags are separate from point tags - hence we can reuse tag `1'
# for our first curve. And as a general rule, elementary entity tags in Gmsh
# have to be unique per geometrical dimension.
gmsh.model.geo.addLine(1, 2, 1)
gmsh.model.geo.addLine(3, 2, 2)
gmsh.model.geo.addLine(3, p4, 3)
gmsh.model.geo.addLine(4, 1, p4)

```
曲線はGmshの第二の要素実体の型で, 曲線の中で直線が最も単純.
内蔵kernelで作られる直線セグメントを作るAPIは同じ変換に従う.
最初の二つ引数はPointタグ(始点と終点)であり, 最後の引数(optionalな一つ)はタグである.

例えば, 次の下のコマンドでは, line 1はpoint1から始まりpoint2で終わる.

注意するべきことは曲線のタグはPointタグとは別である.
つまるところ, 最初の曲線にタグ1を使える.
また, 一般的なルールとして, Gmshの要素実体タグは幾何的の次元に対して一意である必要がある.

```x6.py
# The third elementary entity is the surface. In order to define a simple
# rectangular surface from the four curves defined above, a curve loop has first
# to be defined. A curve loop is defined by an ordered list of connected curves,
# a sign being associated with each curve (depending on the orientation of the
# curve to form a loop). The API function to create curve loops takes a list
# of integers as first argument, and the curve loop tag (which must be unique
# amongst curve loops) as the second (optional) argument:
gmsh.model.geo.addCurveLoop([4, 1, -2, 3], 1)
```
第３の要素実体は表面である.
下で定義されているようにシンプルな四角形の表面は四つの曲線によって定義するために
曲線ループが初めに定義されてなければならない.
曲線ループは並び順を考慮した繋がった曲線のリストで定義される.
符号はそれぞれのカーブにつけられる(ループ形状のカーブの向きから判断)
曲線ループを作るAPI関数は最初の引数に整数のlistを受け取る.
次の引数に曲線ループのタグを受け取る.

```x6.py
# We can then define the surface as a list of curve loops (only one here,
# representing the external contour, since there are no holes--see `t4.py' for
# an example of a surface with a hole):
gmsh.model.geo.addPlaneSurface([1], 1)
```
曲線ループのリストから表面を定義できる.
ここでは, 穴がないため外部の輪郭を表している.
穴がある場合はt4.pyを見てくれ.

```t1.py
# Before they can be meshed (and, more generally, before they can be used by API
# functions outside of the built-in CAD kernel functions), the CAD entities must
# be synchronized with the Gmsh model, which will create the relevant Gmsh data
# structures. This is achieved by the gmsh.model.geo.synchronize() API call for
# the built-in CAD kernel. Synchronizations can be called at any time, but they
# involve a non trivial amount of processing; so while you could synchronize the
# internal CAD data after every CAD command, it is usually better to minimize
# the number of synchronization points.
gmsh.model.geo.synchronize()

```
メッシュを作成する前に(もっというと, 内蔵CAD kernel関数ではないAPI関数を使う前に),
CAD実体はGmshモデルと同期してなければならない.
GmshモデルはGmshデータ構造体との対応を構築する.
同期は内蔵kernelのgmsh.model.synchronize()APIを呼べば良い.

同期はいつでも呼べるが, 大量のプロセスが必要となる.
だから, 理想的にはどのCADコマンドの後にも内部CADデータをを同期することもできるが,
普通は同期するタイミングを最小にすることが推奨される.

```t1.py
# At this level, Gmsh knows everything to display the rectangular surface 1 and
# to mesh it. An optional step is needed if we want to group elementary
# geometrical entities into more meaningful groups, e.g. to define some
# mathematical ("domain", "boundary"), functional ("left wing", "fuselage") or
# material ("steel", "carbon") properties.
#
# Such groups are called "Physical Groups" in Gmsh. By default, if physical
# groups are defined, Gmsh will export in output files only mesh elements that
# belong to at least one physical group. (To force Gmsh to save all elements,
# whether they belong to physical groups or not, set the `Mesh.SaveAll' option
# to 1.) Physical groups are also identified by tags, i.e. stricly positive
# integers, that should be unique per dimension (0D, 1D, 2D or 3D). Physical
# groups can also be given names.
#
# Here we define a physical curve that groups the left, bottom and right curves
# in a single group (with prescribed tag 5); and a physical surface with name
# "My surface" (with an automatic tag) containing the geometrical surface 1:
gmsh.model.addPhysicalGroup(1, [1, 2, 4], 5)
gmsh.model.addPhysicalGroup(2, [1], name = "My surface")
```
この段階で, Gmshで四角形の表面を構成し, メッシュする準備は整ってる.
だけど, geometry実体の要素を意味のある感じにグループ分けしたい時がいずれ訪れる.
"steel"とか"carbon"とかそんなん.
そういうグループはGmshでは"Physical Groups"という.
デフォルトではPhysical groupが定義されていれば, Gmshは少なくとも一つの物理グループに属してる要素は出力ファイルにエクスポートする.
物理グループに属しているかどうかに関係なく、Gmsh にすべての要素を強制的に保存するには、`Mesh.SaveAll' オプションを 1 に設定する.
物理グループは次元ごとに一意である正整数によってによって指定される.
物理グループには名前をつけることもできる.

ここでは左, 下, 右の曲線を一つのグループにグルーピングする物理曲線を定義する.(規定タグの5を使用)
さらに, surface 1を含む物理表面を"My surface"と名付けている.

```t1.py
# We can then generate a 2D mesh...
gmsh.model.mesh.generate(2)

# ... and save it to disk
gmsh.write("t1.msh")
```
2Dメッシュ, 保存はこのようにする.
```t1.py
# To visualize the model we can run the graphical user interface with
# `gmsh.fltk.run()'. Here we run it only if "-nopopup" is not provided in the
# command line arguments:
if '-nopopup' not in sys.argv:
    gmsh.fltk.run()

```
modelをグラフィカルにUI表示するにはgmsh.fltk.run()を使う.
-nopopupをつけるとコマンドライン引数を受け付けない.



```t1.py
# ------------------------------------------------------------------------------
#
#  Gmsh Python tutorial 1
#
#  Geometry basics, elementary entities, physical groups
#
# ------------------------------------------------------------------------------

# The Python API is entirely defined in the `gmsh.py' module (which contains the
# full documentation of all the functions in the API):
import gmsh
import sys

# Before using any functions in the Python API, Gmsh must be initialized:
gmsh.initialize()

# Next we add a new model named "t1" (if gmsh.model.add() is not called a new
# unnamed model will be created on the fly, if necessary):
gmsh.model.add("t1")

# The Python API provides direct access to each supported geometry (CAD)
# kernel. The built-in kernel is used in this first tutorial: the corresponding
# API functions have the `gmsh.model.geo' prefix.

# The first type of `elementary entity' in Gmsh is a `Point'. To create a point
# with the built-in CAD kernel, the Python API function is
# gmsh.model.geo.addPoint():
# - the first 3 arguments are the point coordinates (x, y, z)
# - the next (optional) argument is the target mesh size close to the point
# - the last (optional) argument is the point tag (a stricly positive integer
#   that uniquely identifies the point)
lc = 1e-2
gmsh.model.geo.addPoint(0, 0, 0, lc, 1)

# Note that in addition to the default ``camelCase'' function name `addPoint',
# the Python API also defines a ``snake case'' alias, i.e. `add_point'. You can
# use either interchangeably; all the tutorials are written using the camelCase
# convention for consistency.

# The distribution of the mesh element sizes will be obtained by interpolation
# of these mesh sizes throughout the geometry. Another method to specify mesh
# sizes is to use general mesh size Fields (see `t10.py'). A particular case is
# the use of a background mesh (see `t7.py').
#
# If no target mesh size of provided, a default uniform coarse size will be used
# for the model, based on the overall model size.
#
# We can then define some additional points. All points should have different
# tags:
gmsh.model.geo.addPoint(.1, 0, 0, lc, 2)
gmsh.model.geo.addPoint(.1, .3, 0, lc, 3)

# If the tag is not provided explicitly, a new tag is automatically created, and
# returned by the function:
p4 = gmsh.model.geo.addPoint(0, .3, 0, lc)

# Curves are Gmsh's second type of elementery entities, and, amongst curves,
# straight lines are the simplest. The API to create straight line segments with
# the built-in kernel follows the same conventions: the first 2 arguments are
# point tags (the start and end points of the line), and the last (optional one)
# is the line tag.
#
# In the commands below, for example, the line 1 starts at point 1 and ends at
# point 2.
#
# Note that curve tags are separate from point tags - hence we can reuse tag `1'
# for our first curve. And as a general rule, elementary entity tags in Gmsh
# have to be unique per geometrical dimension.
gmsh.model.geo.addLine(1, 2, 1)
gmsh.model.geo.addLine(3, 2, 2)
gmsh.model.geo.addLine(3, p4, 3)
gmsh.model.geo.addLine(4, 1, p4)

# The third elementary entity is the surface. In order to define a simple
# rectangular surface from the four curves defined above, a curve loop has first
# to be defined. A curve loop is defined by an ordered list of connected curves,
# a sign being associated with each curve (depending on the orientation of the
# curve to form a loop). The API function to create curve loops takes a list
# of integers as first argument, and the curve loop tag (which must be unique
# amongst curve loops) as the second (optional) argument:
gmsh.model.geo.addCurveLoop([4, 1, -2, 3], 1)

# We can then define the surface as a list of curve loops (only one here,
# representing the external contour, since there are no holes--see `t4.py' for
# an example of a surface with a hole):
gmsh.model.geo.addPlaneSurface([1], 1)

# Before they can be meshed (and, more generally, before they can be used by API
# functions outside of the built-in CAD kernel functions), the CAD entities must
# be synchronized with the Gmsh model, which will create the relevant Gmsh data
# structures. This is achieved by the gmsh.model.geo.synchronize() API call for
# the built-in CAD kernel. Synchronizations can be called at any time, but they
# involve a non trivial amount of processing; so while you could synchronize the
# internal CAD data after every CAD command, it is usually better to minimize
# the number of synchronization points.
gmsh.model.geo.synchronize()

# At this level, Gmsh knows everything to display the rectangular surface 1 and
# to mesh it. An optional step is needed if we want to group elementary
# geometrical entities into more meaningful groups, e.g. to define some
# mathematical ("domain", "boundary"), functional ("left wing", "fuselage") or
# material ("steel", "carbon") properties.
#
# Such groups are called "Physical Groups" in Gmsh. By default, if physical
# groups are defined, Gmsh will export in output files only mesh elements that
# belong to at least one physical group. (To force Gmsh to save all elements,
# whether they belong to physical groups or not, set the `Mesh.SaveAll' option
# to 1.) Physical groups are also identified by tags, i.e. stricly positive
# integers, that should be unique per dimension (0D, 1D, 2D or 3D). Physical
# groups can also be given names.
#
# Here we define a physical curve that groups the left, bottom and right curves
# in a single group (with prescribed tag 5); and a physical surface with name
# "My surface" (with an automatic tag) containing the geometrical surface 1:
gmsh.model.addPhysicalGroup(1, [1, 2, 4], 5)
gmsh.model.addPhysicalGroup(2, [1], name = "My surface")

# We can then generate a 2D mesh...
gmsh.model.mesh.generate(2)

# ... and save it to disk
gmsh.write("t1.msh")

# Remember that by default, if physical groups are defined, Gmsh will export in
# the output mesh file only those elements that belong to at least one physical
# group. To force Gmsh to save all elements, you can use
#
# gmsh.option.setNumber("Mesh.SaveAll", 1)

# By default, Gmsh saves meshes in the latest version of the Gmsh mesh file
# format (the `MSH' format). You can save meshes in other mesh formats by
# specifying a filename with a different extension. For example
#
#   gmsh.write("t1.unv")
#
# will save the mesh in the UNV format. You can also save the mesh in older
# versions of the MSH format: simply set
#
#   gmsh.option.setNumber("Mesh.MshFileVersion", x)
#
# for any version number `x'. As an alternative, you can also not specify the
# format explicitly, and just choose a filename with the `.msh2' or `.msh4'
# extension.

# To visualize the model we can run the graphical user interface with
# `gmsh.fltk.run()'. Here we run it only if "-nopopup" is not provided in the
# command line arguments:
if '-nopopup' not in sys.argv:
    gmsh.fltk.run()

# Note that starting with Gmsh 3.0, models can be built using other geometry
# kernels than the default "built-in" kernel. To use the OpenCASCADE CAD kernel
# instead of the built-in kernel, you should use the functions with the
# `gmsh.model.occ' prefix.
#
# Different CAD kernels have different features. With OpenCASCADE, instead of
# defining the surface by successively defining 4 points, 4 curves and 1 curve
# loop, one can define the rectangular surface directly with
#
# gmsh.model.occ.addRectangle(.2, 0, 0, .1, .3)
#
# After synchronization with the Gmsh model with
#
# gmsh.model.occ.synchronize()
#
# the underlying curves and points could be accessed with
# gmsh.model.getBoundary().
#
# See e.g. `t16.py', `t18.py', `t19.py' or `t20.py' for complete examples based
# on OpenCASCADE, and `examples/api' for more.

# This should be called when you are done using the Gmsh Python API:
gmsh.finalize()
```
