---
title: "Index(1) B-Tree & B+Tree"
categories:
    - database
tags:
    - cs
    - database
    - computer structure
permalink: /database/:title
toc:
    true
---

<br>
<br>

# 앞서...

데이터베이스 인덱스, 클러스터형 인덱스, 비클러스터형 인덱스를 공부하는중에, 정말 아름다운 유튜브 영상을 보게 되었습니다.

화려한 기술, 복잡한 로직에 관한 영상은 아니지만, 인덱스를 사용해야 하는 이유, 어떻게 B-Tree가 발전해왔나와 같은 근본적인 내용을 다루는 영상입니다.

해당 포스트는 이 영상의 번역본에 불과하므로 해당 영상을 시청하시는 것이 훨씬 도움이 될 것 같습니다.

영어로 되어있지만 차근차근 친절히 설명하고 크게 어려운 용어도 없으니 한번 시청하시는 것을 추천드립니다.

[[YouTube] B Trees and B+ Trees. How they are useful in Databases - Abdul Bari](https://www.youtube.com/watch?v=aZjYr87r1b8)

<br>
<br>

# 인덱스란

인덱스란, 추가 저장공간과 연산 작업을 사용하여 데이터베이스 테이블의 탐색 성능을 높이는데 사용됩니다.

이 인덱스를 관리하는 자료구조로 보통 B Tree 계열의 자료구조를 사용합니다.

B tree family들에 해대 이야기 하기 전, 데이터베이스와 관련된 다른 중요한 개념들을 먼저 알아보겠습니다.

<br>
<br>

# 1. Disk Structure

데이터베이스는 기본적으로 디스크에 상주되어있습니다. 데이터베이스 자체가 삭제되거나 물리적 저장 공간이 파괴되지 않는 이상 데이터는 유지되어야 하므로 디스크가 사용됩니다.

디스크는 데이터를 원판에 저장하고 원판을 돌리며 arm의 head가 위치한 점의 데이터를 읽는 형식으로 데이터를 읽습니다.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/database/Index(1)_B-Tree_and_B+Tree/i1_disk.png" width="70%" height="70%">

디스크 구조의 간단한 개념과 용어만 짚고 넘어가겠습니다.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/database/Index(1)_B-Tree_and_B+Tree/i2_sector,track,block,offset.png">

하나의 원판은 sector 라는 개념적인 부분과 track이라는 물리적인 부분으로 구성되어있습니다.

하나의 sector 안에서 track으로 구분된 하나의 조각은 block 이라고 합니다.

또 block은 각 바이트마다 주소를 저장할 수 있는데 이 바이트의 번호를 offset이라고 합니다.

즉, sector 번호, track 번호를 사용하여 block을 알아내고, 해당 block의 offset을 탐색하며 데이터를 읽습니다.

이 주소를 알아내기 위해 원판을 돌리며 sector와 offset을 조정하고, arm을 앞뒤로 움직이며 track을 탐색하는 물리적인 행위가 디스크 탐색에 소모되는 시간의 대부분을 차지합니다.

탐색하는 block의 수를 줄인다면 탐색 시간을 줄일 수 있을 것입니다.

이를 위해 생긴 것이 index 입니다.

<br>
<br>

# 2. Index

앞서 말한것과 같이 탐색 시간을 줄이기 위해 생긴 것이 index입니다.

인덱스가 어떤 식으로 작동되는지 예시를 통해 알아보겠습니다.

다음과 같은 구조를 가지는 Employee 테이블을 만들겠습니다.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/database/Index(1)_B-Tree_and_B+Tree/t1_Employee_table.png" width="50%" height="50%">

이 테이블에 저장될 하나의 row는 128B(10+50+10+8+50)의 크기를 차지할 것입니다.

앞서 디스크는 block으로 이루어져있다고 했는데, 이 block의 크기가 512B 라고 가정해보겠습니다.

![]({{ site.url }}{{ site.baseurl }}/assets/images/database/Index(1)_B-Tree_and_B+Tree/i3_four_records_in_a_block.png)

Block의 크기는 512B이므로 하나의 block에는 총 네개(512/128)의 record가 저장될 수 있습니다.

다음과 같이 Employee 테이블에 100개의 row가 저장되어 있다고 하겠습니다.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/database/Index(1)_B-Tree_and_B+Tree/t2_100_employees.png" width="50%" height="50%">

그렇다면 해당 테이블의 데이터들은 총 25개(100/4)의 block을 차지하고 있는 셈입니다.

즉, 이 Employee 테이블에서 데이터를 검색하고자 한다면 총 25개의 block을 탐색해야 합니다.

25개의 block을 모두 탐색하지 않고 더 적은 수의 block을 탐색하기 위해 생긴 것이 index 입니다.

![]({{ site.url }}{{ site.baseurl }}/assets/images/database/Index(1)_B-Tree_and_B+Tree/i4_dense_index_table.png)

eid를 인덱스로 설정한다면 인덱스 테이블에는 eid와 포인터가 저장됩니다. 포인터는 실제 데이터 레코드를 가리키고 있습니다. 해당 인덱스 테이블은 밀집하게 하나의 데이터 레코드씩을 가리키고 있으므로 dense index 테이블이라 합니다. (기억해 두세요!)

그렇다면 이 인덱스는 어디에 저장될까요? 실제 데이터 레코드와 마찬가지로 디스크에 저장됩니다.

위 인덱스 테이블은 eid와 포인터가 저장된다 했는데, 각각 10B, 6B 크기를 가진다고 하겠습니다. 즉 하나의 인덱스 레코드는 16B의 크기를 가지는 것입니다. 하나의 인덱스 레코드가 16B의 크기를 가지니 하나의 블럭에 총 32개(512/16)dml 인덱스 레코드가 저장될 수 있습니다.

해당 인덱스 테이블의 각 row는 실제 데이터 레코드와 일대일 매칭되므로 총 100개의 row를 가집니다. 따라서, 100개의 인덱스 레코드를 저장하는데에는 네개(approx 100/16)의 block이 필요로 합니다.

인덱스를 사용했더니 25개의 block을 모두 탐색하지 않고 다섯개(인덱스 레코드용 네개 + 어떤 테이블인지 알기 위해 사용되는 block 한개)의 block만 탐색해도 되게 되었습니다!

<br>
<br>

# 3. Multi-level index

인덱스를 사용하여 탐색 해야 하는 block 수를 크게 줄인 것을 확인하였습니다. 그런데 위 예시에서 인덱스와 실제 데이터 레코드는 일대일 매칭되어있습니다. 결국 데이터가 늘어남에 따라 인덱스의 수도 정비례하게 늘어날 것이고, 탐색해야 하는 block의 수도 비례하게 늘어날 것입니다. 100개의 데이터를 저장하는데 네개의 block이 필요했으므로 1000개의 데이터를 저장하는데에는 40개의 block이 인덱스를 위해 필요할 것입니다.

이렇게 데이터 크기에 따라 수가 증가하는 인덱스를 가리키는 인덱스를 만들어보겠습니다. 이를 multi-level index라고 합니다.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/database/Index(1)_B-Tree_and_B+Tree/i5_multi-level_index.png" width="50%" height="50%">

실제 데이터 레코드를 가리키는 인덱스를 1차 인덱스, 1차 인덱스를 가리키는 인덱스를 2차 인덱스라고 하겠습니다.(1차, 2차 인덱스는 실제로 사용되는 용어가 아니고 설명의 용이를 위해 만들어낸 용어입니다.)

2차 인덱스 또한 1차 인덱스와 일대일로 생성한다면 데이터만 차지할 뿐 아무런 도움이 되지 않을 것입니다. 1차 인덱스를 특정 크기의 범위로 나누고 해당 범위를 가리키는 2차 인덱스를 만들어보겠습니다. 2차 인덱스가 가리킬 범위의 크기를 32라고 한다면, 2차 인덱스의 1번 row는 1차 인덱스의 1~32번 row를 가리킬 것이고, 2차 인덱스의 2번 row는 1차 인덱스의 33~64번 row를 가리킬 것입니다.

2차 인덱스 row 또한 1차 인덱스 row와 마찬가지로 id(1차 인덱스 블럭의)와 포인터만 가지므로 16B를 차지하게 됩니다.

2차 인덱스는 32개씩 묶인 1000개의 row에 대한 포인터를 가지므로 32개(approx 1000/32)의 row를 가지고, 이는 한개(16*32/512)의 block을 필요로 합니다.

2차 인덱스에서 1차 인덱스를 탐색하기 위해서는 두개(한개 + 어떤 1차 인덱스 테이블인지 알기 위한 한개)의 블럭 을 탐색해야 합니다.

이렇게 두 단계의 인덱스 테이블을 사용하여, 2차 인덱스 테이블에서 두개 block 탐색, 1차 인덱스 테이블에서 한개 block 탐색(32개의 row만 탐색하면 되므로), 총 두개의 block만 탐색해도 되게 되었습니다!

인덱스를 관리하는데 효과적인 Multi-level index는 어떤 자료구조로 구성하면 효과적일까요?

<br>
<br>

# 4. M-way Search Tree

M-way Search Tree에 대해 이야기 하기 전에 잠깐 Binary Search Tree(이하 BST)에 대해 이야기해보겠습니다.

![]({{ site.url }}{{ site.baseurl }}/assets/images/database/Index(1)_B-Tree_and_B+Tree/i6_BST.png)

위와 같이 부모 노드보다 작은 값은 부모 노드의 왼쪽에, 부모 노드보다 큰 값은 부모 노드의 오른쪽에 위치한 트래 구조를 BST 라고 합니다. BST는 하나의 노드당 최대 두개의 자식만을 가질 수 있습니다. 이 BST는 M-way Search Tree의 일종입니다.

그렇습니다. 이러한 형식으로 최대 M개의 자식을 가지는 트리를 M-way Search Tree라고 합니다. BST는 2-way Search Tree가 되는 것입니다.

M-way Search Tree는 하나의 노드당 최대 M-1개의 데이터를 가질 수 있고, 이 노드 안의 데이터들은 항상 오름차순으로 정렬되어 있습니다.

부모 노드에 10, 20 이 저장되어있는 3-way Search Tree를 생각해보겠습니다.

![]({{ site.url }}{{ site.baseurl }}/assets/images/database/Index(1)_B-Tree_and_B+Tree/i7_3-way_Search_Tree.png)

10보다 작은 데이터는 첫번째 자식 노드에, 10보다 크고 20보다 작은 데이터는 두번째 자식 노드에, 20보다 큰 데이터는 세번째 자식 노드에 저장됩니다. 이처럼 M-way Search Tree의 노드에는 M-1인 두개의 데이터가 저장되고, M개인 세개의 자식을 위한 포인터가 저장됩니다.

그렇다면 이 M-way Search Tree를 사용하여 multi-level index를 효과적으로 구현할 수 있을까요?

4-way Search Tree를 사용한 예시를 확인해보겠습니다.

![]({{ site.url }}{{ site.baseurl }}/assets/images/database/Index(1)_B-Tree_and_B+Tree/i8_4-way_Search_Tree.png)

이처럼 하나의 노드에는 세개(M-1)의 인덱스가 저장되고, 네개(M)의 자식 포인터가 있습니다. 그런데 우리가 알고 싶은 것은 실제 데이터 레코드이고, 인덱스는 이 데이터 레코드를 가리키는 포인터를 가지고 있어야 합니다.

즉, 세개의 인덱스가 가리키는 데이터 레코드의 포인터 또한 가지고 있어야 합니다.

![]({{ site.url }}{{ site.baseurl }}/assets/images/database/Index(1)_B-Tree_and_B+Tree/i9_with_record_pointer.png)

위 노드 형태가 M-way Search Tree 노드의 형태이고, 이후 확인할 B-Tree의 형태이기도 합니다.

그렇다면 M-way Search Tree는 multi-level index를 관리하기 효과적인 자료구조일까요?

M-way Search Tree는 치명적인 약점이 있습니다. 바로 트리 구조를 짜는데 아무런 제약 조건이 없다는 점입니다.

10-way Search Tree를 예시로 보겠습니다.

빈 10-way Search Tree에 10, 20, 30이라는 데이터를 순차적으로 넣겠습니다.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/database/Index(1)_B-Tree_and_B+Tree/i10_10-way_Search_Tree(1).png" width="50%" height="50%">

10을 삽입하였습니다.

![]({{ site.url }}{{ site.baseurl }}/assets/images/database/Index(1)_B-Tree_and_B+Tree/i11_10-way_Search_Tree(2).png)

20은 10보다 큰 값이니 10 오른쪽의 포인터가 가리키는 자식 노드를 만들고 20을 넣었습니다.

![]({{ site.url }}{{ site.baseurl }}/assets/images/database/Index(1)_B-Tree_and_B+Tree/i12_10-way_Search_Tree(3).png)

30 역시 20보다 큰 값이므로 20 오른쪽의 포인터가 가리키는 자식 노드를 만들고 30을 넣었습니다.

보시다시피 당연히 이 구조는 비효율적입니다.

아래와 같이 하나의 노드에 10, 20, 30 데이터를 모두 넣는 것이 효율적이지만, M-way Search Tree는 구조를 생성하는데 아무런 제약조건이 없으므로 위 구조도 문제 없는 M-way Search Tree입니다.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/database/Index(1)_B-Tree_and_B+Tree/i13_efficient_10-way_Search_Tree.png" width="70%" height="70%">

불필요하게 자식 노드를 생성하여 깊이가 늘어가고, 탐색 속도에도 큰 영향을 끼치게 될 것입니다.

M-way Search Tree는 이처럼 제약 조건이 없어 비효율적으로 구성될 수 있다는 약접을 가지고 있고, multi-level index를 관리할 자료구조로는 적합해 보이지 않습니다.

(M-way Search Tree는 사실 인터페이스에 더 가까운 느낌입니다.)

이 약점을 보완할 제약을 가지는 자료구조가 바로 B-Tree 입니다.

<br>
<br>

# 5. B-Tree

(비 트리라고 읽습니다.)

앞서 말한것과 같이 B-Tree는 M-way Search Tree와 동일한 개념을 가지지만, 효율적인 구조 생성을 위한 제약 조건을 가지고 있습니다.

M-way Search Tree가 B-Tree가 되기 위해 필요한 조건은 다음과 같습니다.

- 첫번째 조건: B-Tree의 노드는 최소 ceil(M/2)개의 데이터를 가져야 자식 노드를 만들 수 있습니다.\
즉, 10차 B-Tree의 노드는 최소 다섯개의 데이터를 가지고 있어야 다음 노드를 만들 수 있습니다.

- 두번째 조건: Root 노드가 leaf노드가 아니라면, 최소 두개의 자식 노드를 가지고 있어야 합니다.

- 세번째 조건: 모든 leaf 노드는 같은 level에 있습니다.

또한 B-Tree의 insertion은 bottom-up 형식입니다.

예를 통해 확인해보겠습니다.

4차 B-Tree에 10, 20, 40, 50을 순차적으로 삽입해보겠습니다.

![]({{ site.url }}{{ site.baseurl }}/assets/images/database/Index(1)_B-Tree_and_B+Tree/i14_self_balancing(1).png)

이처럼 적절한 노드에 더이상 데이터를 넣을 자리가 없다면 적절한 데이터를 부모 노드로 올립니다.

![]({{ site.url }}{{ site.baseurl }}/assets/images/database/Index(1)_B-Tree_and_B+Tree/i15_self_balancing(2).png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/database/Index(1)_B-Tree_and_B+Tree/i16_self_balancing(3).png)

예시에서는 새로운 부모 노드를 만들 때 두개의 가능 후보군 중 오른쪽 데이터를 선택했습니다. 사실 왼쪽을 선택하든 오른쪽은 선택하든 아무런 상관이 없고 이는 구현하는 프로그래머 마음입니다. 위의 예시처럼 오른쪽 데이터를 올린 경우를 right biased라고 합니다.

또, 데이터를 삽입해나감에 따라 아래에서 위로 트리가 솟아나며 깊이가 깊어지는 것을 볼 수 있습니다. 이때문에 B-Tree의 삽입 형태가 bottom-up 형태라고 한 것입니다.

이처럼 self balancing 기능이 있는 B-Tree는 인덱스를 관리하는데 안성맞춤입니다.

이전에 보았던 M-way Search Tree의 노드 형태를 다시 확인해보겠습니다.

![]({{ site.url }}{{ site.baseurl }}/assets/images/database/Index(1)_B-Tree_and_B+Tree/i18_M-way_Search_Tree_node.png)

이와 같이 노드에는 M-1개의 데이터(인덱스)가 있고, 해당 인덱스가 가리키는 실제 레코드 포인터가 존재합니다.

B-Tree의 모든 노드들은 아래의 이미지와 같이 데이터 레코드를 가리키는 포인터를 가지고 있는 셈입니다.

![]({{ site.url }}{{ site.baseurl }}/assets/images/database/Index(1)_B-Tree_and_B+Tree/i17_B-Tree_record_pointer.png)

앞서 보았던 multi-level index의 dense함과는 거리가 있는 sparse함이 보이지만, B-Tree는 index 관리에 사용되는 자료구조입니다.

<br>
<br>

# 6. B+Tree

(비 플러스 트리로 읽습니다.)

B+Tree 또한 B-Tree와 동일한 구조를 가지고 있습니다.

단지 다른 점은, 모든 노드에서 데이터 레코드를 가리키는 포인터를 가지는 B-Tree와는 다르게, 오직 leaf 노드에만 데이터 레코드를 가리키는 포인터를 가진다는 점입니다.

결국 B+Tree는 리프 노드에 모든 인덱스를 가지고 있어야 하므로, 부모 노드의 데이터들이 모두 리프 노드들에 포함되어 있어야 합니다.

![]({{ site.url }}{{ site.baseurl }}/assets/images/database/Index(1)_B-Tree_and_B+Tree/i19_B+Tree.png)

또한, B+Tree의 모든 leaf 노드는 linked list 구조로 되어있습니다.

Leaf 노드의 인덱스들은 모든 인덱스들이므로 dense한 index 형태를 가집니다.

![]({{ site.url }}{{ site.baseurl }}/assets/images/database/Index(1)_B-Tree_and_B+Tree/i20_linked_leaf_node_list.png)

그토록 찾던 multi-level index와 동일한 형태를 띄고 있는 것을 확인할 수 있습니다.

Leaf 노드들은 multi-level index에서의 1차 인덱스, 위의 레벨 노드들은 2차, 3차,... 인덱스가 되는 것입니다.

대부분의 RDBMS들이 이 B+Tree를 인덱스 구조에 사용하고 있습니다.

<br>
<br>

# Clustered index & Non-clustered index

다음 포스트에서는 B Tree 계열로 구현하는 clustered index와 non-clustered index의 특징과 차이점을 정리해보겠습니다.

<br>
<br>
<br>
<br>

틀린 정보가 있다면 댓글로 말씀 부탁드리겠습니다!