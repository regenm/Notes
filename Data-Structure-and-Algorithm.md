---
title: Data Structure and Algorithm
date: 2023-11-12 21:58:40
tags: 笔记
---

# Read me:

​	本部分笔记为散记，用于记录我认为的疑难杂点。

# Start

* **哈夫曼树（最优二叉树）**

> ​		哈夫曼算法：
>
> 1. 构造 n 棵二叉树森林，每一个都是带权值的根节点。
> 2. 选择权值最小的两棵树作为左右子树，其根节点的权值为左右子树权值之和。
> 3. 删除这两棵树，将新的树加入森林。
> 4. 重复操作到只剩下一棵树为止。

```c++
/*
*	基于结构体数组的哈曼树
*	C++
*/
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef double DataType; //结点权值的数据类型

typedef struct HTNode //单个结点的信息
{
	DataType weight; //权值
	int parent; //父节点
	int lc, rc; //左右孩子
}*HuffmanTree;

typedef char **HuffmanCode; //字符指针数组中存储的元素类型
						  //用于存储哈夫曼编码

//在下标为1到i-1的范围找到权值最小的两个值的下标，其中s1的权值小于s2的权值
void Select(HuffmanTree& HT, int n, int& s1, int& s2)
{
	int min;
	//找第一个最小值
	for (int i = 1; i <= n; i++)
	{
		if (HT[i].parent == 0)
		{
			min = i;
			break;
		}
	}
	for (int i = min + 1; i <= n; i++)
	{
		if (HT[i].parent == 0 && HT[i].weight < HT[min].weight)
			min = i;
	}
	s1 = min; //第一个最小值给s1
	//找第二个最小值
	for (int i = 1; i <= n; i++)
	{
		if (HT[i].parent == 0 && i != s1)
		{
			min = i;
			break;
		}
	}
	for (int i = min + 1; i <= n; i++)
	{
		if (HT[i].parent == 0 && HT[i].weight < HT[min].weight&&i != s1)
			min = i;
	}
	s2 = min; //第二个最小值给s2
}

//构建哈夫曼树
void CreateHuff(HuffmanTree& HT, DataType* w, int n)
{
	int m = 2 * n - 1; //哈夫曼树总结点数
    
	HT = (HuffmanTree)calloc(m + 1, sizeof(HTNode)); //开m+1个HTNode，因为下标为0的HTNode不存储数据
    
	for (int i = 1; i <= n; i++)
	{
		HT[i].weight = w[i - 1]; //赋权值给n个叶子结点
	}
    
	for (int i = n + 1; i <= m; i++) //构建哈夫曼树
	{
		//选择权值最小的s1和s2，生成它们的父结点
		int s1, s2;
		Select(HT, i - 1, s1, s2); //在下标为1到i-1的范围找到权值最小的两个值的下标，其中s1的权值小于s2的权值
		HT[i].weight = HT[s1].weight + HT[s2].weight; //i的权重是s1和s2的权重之和
		HT[s1].parent = i; //s1的父亲是i
		HT[s2].parent = i; //s2的父亲是i
		HT[i].lc = s1; //左孩子是s1
		HT[i].rc = s2; //右孩子是s2
	}
    
	//打印哈夫曼树中各结点之间的关系
	printf("哈夫曼树为:>\n");
	printf("下标   权值     父结点   左孩子   右孩子\n");
	printf("0                                  \n");
	for (int i = 1; i <= m; i++)
	{
		printf("%-4d   %-6.2lf   %-6d   %-6d   %-6d\n", i, HT[i].weight, HT[i].parent, HT[i].lc, HT[i].rc);
	}
	printf("\n");
}

//生成哈夫曼编码
void HuffCoding(HuffmanTree& HT, HuffmanCode& HC, int n)
{
	HC = (HuffmanCode)malloc(sizeof(char*)*(n + 1)); //开n+1个空间，因为下标为0的空间不用
	char* code = (char*)malloc(sizeof(char)*n); //辅助空间，编码最长为n(最长时，前n-1个用于存储数据，最后1个用于存放'\0')
	code[n - 1] = '\0'; //辅助空间最后一个位置为'\0'
	for (int i = 1; i <= n; i++)
	{
		int start = n - 1; //每次生成数据的哈夫曼编码之前，先将start指针指向'\0'
		int c = i; //正在进行的第i个数据的编码
		int p = HT[c].parent; //找到该数据的父结点
		while (p) //直到父结点为0，即父结点为根结点时，停止
		{
			if (HT[p].lc == c) //如果该结点是其父结点的左孩子，则编码为0，否则为1
				code[--start] = '0';
			else
				code[--start] = '1';
			c = p; //继续往上进行编码
			p = HT[c].parent; //c的父结点
		}
		HC[i] = (char*)malloc(sizeof(char)*(n - start)); //开辟用于存储编码的内存空间
		strcpy(HC[i], &code[start]); //将编码拷贝到字符指针数组中的相应位置
	}
	free(code); //释放辅助空间
}

//主函数
int main()
{
	int n = 0;
	printf("请输入数据个数:>");
	scanf("%d", &n);
	DataType* w = (DataType*)malloc(sizeof(DataType)*n);
	if (w == NULL)
	{
		printf("malloc fail\n");
		exit(-1);
	}
	printf("请输入数据:>");
	for (int i = 0; i < n; i++)
	{
		scanf("%lf", &w[i]);
	}
	HuffmanTree HT;
	CreateHuff(HT, w, n); //构建哈夫曼树

	HuffmanCode HC;
	HuffCoding(HT, HC, n); //构建哈夫曼编码

	for (int i = 1; i <= n; i++) //打印哈夫曼编码
	{
		printf("数据%.2lf的编码为:%s\n", HT[i].weight, HC[i]);
	}
	free(w);
	return 0;
}


```











* **并查集**

​	 	主要用于解决一些**元素分组**的问题。也可以用来判断图的连通性，它管理一系列**不相交的集合**，并支持两种操作：

- **合并**（Union）：把两个不相交的集合合并为一个集合。
- **查询**（Find）：查询两个元素是否在同一个集合中。

```c
int find(int x)	//最简单版本
{
    /*
    *当数据很多时，查找会很耗费时间
    *
    */
    if(father[x]!=x) father[x]=find(father[x]);
    return father[x];
}

int find(int x){           //查找根节点的函数 
    /*
    *	升级版
    *	带路径压缩功能
    */
    int a;
	a=x;
	while(x!=father[x])x=father[x];
	
	while(a!=father[a]){    //路径压缩 
		int z=a;
		a=father[a];
		father[z]=x;
	}
	return x;
}
void Union(int a,int b){ 
    						//合并两个集合
    int fx=find(a);
    int fy=find(b);
    father[fx]=fy;
}

int judgeConnect(){
    						//判断连通性
    int i,k=0;
    for(i=1;i<=vertex;i++)
        if(father[i]==i) k++;
        if(k==1) return 1;
    else return 0;
    
}
```

* **AOE图**

> *concept:*
>
> 1、**最早发生**时间：从前往后，前驱结点到当前结点所需时间，取**最大值**；
>
> 2、**最迟发生**时间：从后往前，后继结点的最迟时间减去边权的值，取**最小值**；
>
> **结束节点的最早发生时间和最迟发生时间相同。**
>
> 3、关键路径：最早发生时间和最迟发生时间相同的结点叫做关键路径上的结点；
>
> 4、**最早开始**时间：等于当前边起始节点的最早发生时间；
>
> 5、**最晚开始**时间：等于当前便指向结点的最迟时间减去当前边的权值；
>
> 6、**最早完工**时间：等于当前边指向结点的最早发生时间；
>
> 7、**最晚完工**时间：等于当前边指向结点的最迟发生时间；

* **图的链式存储结构**

```c
#include <stdio.h>
#include <stdlib.h>
#define MVNum 100                                 //最大顶点数 
typedef struct ArcNode{                        //表结点 
    int adjvex;                                    //邻接点的位置 
    struct ArcNode * nextarc;      //指向下一个表结点的指针 
  }ArcNode; 
  
typedef struct VNode{ 
   char data;                                    //顶点信息 
   ArcNode * firstarc;         //指向第一个表结点的指针 
}VNode, AdjList[MVNum];                 //AdjList表示邻接表类型 

typedef struct{ 
    AdjList vertices;              //头结点数组
    int vexnum, arcnum;     //图的当前顶点数和边数 
}ALGraph; 

void CreatMGraph(ALGraph *G);/* 创建图 */
void printGraph(ALGraph G);/*输出图 */
int main()
{
    ALGraph G;
    CreatMGraph(&G);
    printGraph(G);
    return 0;
}
void CreatMGraph(ALGraph *G)
{
    int i,j,k;
    ArcNode *s;
    scanf("%d%d",&G->vexnum,&G->arcnum);
    getchar();
    for(i=0;i<G->vexnum;i++)
         scanf("%c",&G->vertices[i].data);

    for(i=0;i<G->vexnum;i++)
         G->vertices[i].firstarc=NULL;

    for(k=0;k<G->arcnum;k++) {  
        scanf("%d%d",&i,&j);    
        s=(ArcNode*)malloc(sizeof(ArcNode));
        s->adjvex=j;
        s->nextarc=G->vertices[i].firstarc;

        G->vertices[i].firstarc=s;   

        s=(ArcNode*)malloc(sizeof(ArcNode));
        s->adjvex=i;

        s->nextarc=G->vertices[j].firstarc;;
        G->vertices[j].firstarc=s;  
    }
}
void printGraph(ALGraph G)
{
    int i,j;
    ArcNode *p;
    for(i=0;i<G.vexnum;i++)
    {
       printf("%c:",G.vertices[i].data);
       for(p=G.vertices[i].firstarc;p;p=p->nextarc)
           printf(" %c",G.vertices[p->adjvex].data);
       printf("\n");
    }
}
```

* **二叉树：先序遍历**

```c
int PreOrder(BiTree T){
   if(T!=NULL){
      visit(T);                 //访问根结点
       
      PreOrder(T->lchild);      //递归遍历左子树
      PreOrder(T->rchild);      //递归遍历右子树
   }
    else{
        return 0;				//遍历失败，根节点空
    }
    return 1;
}
```

**二叉树：顺序存储**

​	存储按照完全二叉树来（遇到空节点则 赋值 isEmpty 为 true）

```c
#define MaxSize 100

struct TreeNode{
   ElemType value; //结点中的数据元素
   bool isEmpty;   //结点是否为空
}

int main(){
   TreeNode t[MaxSize];
   for (int i=0; i<MaxSize; i++){
      t[i].isEmpty = true;
   }
}

```



**二叉树：中序遍历**

```c
int ShowZhongXu(BitTree T)			//		先序遍历二叉树
{
	if(T==NULL)						//	递归中遇到NULL，返回上一层节点
	{
		return 0;
	}
	else {
	ShowZhongXu(T->lchild);			//	递归遍历左子树
	printf("%d ",T->data);
	ShowZhongXu(T->rchild);			//	递归遍历右子树
    }
    return 1;
}
```

**二叉树：后序遍历**

```c
int ShowZhongXu(BitTree T)			//		先序遍历二叉树
{
	if(T==NULL)						//	递归中遇到NULL，返回上一层节点
	{
		return 0;					// 空根
	}
	else {
	ShowZhongXu(T->lchild);			//	递归遍历左子树
	ShowZhongXu(T->rchild);			//	递归遍历右子树
    printf("%d ",T->data);			
    }
    return 1;
}
```

- **二叉树：层序遍历**

* 使用队列

```c
void LevelOrder(BiTree T){
   LinkQueue Q;
   InitQueue (Q);          //初始化辅助队列
    
   BiTree p;
   EnQueue(Q,T);           //将根节点入队
    
   while(!isEmpty(Q)){     //队列不空则循环
      DeQueue(Q,p);        //队头结点出队
      visit(p);            //访问出队结点
      if(p->lchild != NULL)
         EnQueue(Q,p->lchild);   //左孩子入队
      if(p->rchild != NULL)
         EnQueue(Q,p->rchild);   //右孩子入队
   }
}
```

* **线索二叉树**

> 将空的指针域用以指向前驱后继节点。
>
> 遵循：
>
> - ltag==0，指向左孩子；ltag==1，指向前驱结点
> - rtag==0，指向右孩子；rtag==1，指向后继结点

		1. 二叉树的线索化

```c
//中序线索化
void inOrderThreadTree(Node* node)
{
	//如果当前结点为NULL 直接返回
	if (node == NULL) {
		return;
	}
	//先处理左子树
	inOrderThreadTree(node->left_node);
	if (node->left_node == NULL)
	{
		//设置前驱结点
		node->left_type = 1;
		node->left_node = pre;
	}
	//如果结点的右子节点为NULL 处理前驱的右指针
	if (pre !=NULL && pre->right_node == NULL)
	{
		//设置后继
		pre->right_node = node;
		pre->right_type = 1;
	}
	//每处理一个节点 当前结点是下一个节点的前驱
	pre = node;
	//最后处理右子树
	inOrderThreadTree(node->right_node);
}

```

2. 中序遍历线索二叉树

```c
void inOrderTraverse(Node* root)
{
	//从根节点开始先找到最左边
	if (root == NULL)
	{
		return;
	}
	Node* temp = root;
	//先找到最左边结点 然后根据线索化直接向右遍历
	while (temp != NULL && temp->left_type == 0)
	{
		temp = temp->left_node;
	}
	while (temp != NULL)
	{
		//输出
		temp = temp->right_node;
	}
}
```

* **折半搜索**

> 因为折半查找需要方便地定位查找区域，所以它要求线性表必须具有随机存取的特性。因此，该查找法仅适合于顺序存储结构，不适合于链式存储结构，且要求元素按关键字有序排列。

```c
#include<stdio.h>

int BinSearch(int array[], int x, int n);

int main()
{
    int arr[10];//0 1 2 3 4 5 6 7 8 9 

    int num;
    for(int i=0;i<10;i++)
    {
        arr[i]=i;
    }
    int re=BinSearch(arr,9,10);
    printf("下标：%d\n",re);
    return 0;
}
								//按折半查找法查找值为x的数组元素，若找到则返回x在数组中的下标位置，否则返回-1
int BinSearch(int array[], int x, int n)
{
    int low = 0, high = n-1, mid;//区间左端点low置为0，右端点high置为n-1
    int num=0;
    while (low <= high)			//若左端点小于等于右端点，则继续查找
    {
        num++;
        mid = low + (high - low) / 2;//取数据区间的重点
        printf("%d\n",mid);
        if (x > array[mid])
            low = mid + 1;			//若x>array[mid]，则修改区间的左端点
        else if (x < array[mid])
            high = mid - 1;			//若x<array[mid]，则修改区间的右端点
        else{
            printf("次数：%d\n",num);
            return mid;				//若找到，则返回下标值mid
            }
    }
    return -1;						//循环结束仍未找到，则返回值-1
}
```

* **插值查找**

> 类似于折半搜索，只是mid的计算方法不一样
>
> 比较元素的下标：
>
> Mid = Begin + ( (End - Begin) / (A[End] - A[Begin]) ) * (X - A[Begin])
>
> >式子中，各部分的含义分别是：
> >
> >Mid：计算得出的元素的位置；
> >
> >End：搜索区域内最后一个元素所在的位置；
> >
> >Begin：搜索区域内第一个元素所在的位置；
> >
> >X：要查找的目标元素；
> >
> >A[]：表示整个待搜索序列

C语言实现过程

​	**递归算法**

```c
#include <stdio.h>
//实现插值查找算法，ele 表示要查找的目标元素，[begin,end] 指定查找区域
int interpolation_search(int* arr, int begin, int end, int ele) {
    int mid = 0;
    //如果[begin,end] 不存在，返回 -1
    if (begin > end) {
        return -1;
    }
    //如果搜索区域内只有一个元素，判断其是否为目标元素
    if (begin == end) {
        if (ele == arr[begin]) {
            return begin;
        }
        //如果该元素非目标元素，则查找失败
        return -1;
    }
    // 找到"中间元素"所在的位置
    mid = begin + ((end - begin) / (arr[end] - arr[begin]) * (ele - arr[begin]));
    //递归的出口
    if (ele == arr[mid]) {
        return mid;
    }
    //比较 ele 和 arr[mid] 的值，缩小 ele 可能存在的区域
    if (ele < arr[mid]) {
        //新的搜索区域为 [begin,mid-1]
        return interpolation_search(arr, begin, mid - 1, ele);
    }
    else {
        //新的搜索区域为 [mid+1,end]
        return interpolation_search(arr, mid + 1, end, ele);
    }
}
int main()
{
    int arr[10] = { 1,2,3,4,5,6,7,8,9,10 };
    //输出元素 2 所在位置的下标
    int pos = interpolation_search(arr, 0, 9, 2);
    if (pos != -1) {
        printf("%d", interpolation_search(arr, 0, 9, 2));
    }
    else {
        printf("查找失败");
    }
    return 0;
}
```

* **二叉排序树**

> 1. 若左子树非空，则左子树上所有结点的值均小于根结点的值。
> 2. 若右子树非空，则右子树上所有结点的值均大于根结点的值。
> 3. 左、右子树也分别是一棵二叉排序树。

```c
/*二叉树的二叉链表结点结构定义*/
typedef struct BiTNode
{
	int data;	//结点数据
	struct BiTNode *lchild, *rchild;	//左右孩子指针
} BiTNode, *Bitree;

/*
递归查找二叉排序树T中是否存在key
指针f指向T的双亲，其初始调用值为NULL
若查找成功，则指针p指向该数据元素结点，并返回TRUE
否则指针p指向查找路径上访问的最后一个结点并返回FALSE
*/
bool SearchBST(BiTree T, int key, BiTree f, BiTree *p){
	if(!T){
		*p = f;
		return FALSE;
	}else if(key == T->data){
		//查找成功
		*p = T;
		return TRUE;
	}else if(key < T->data){
		return SearchBST(T->lchild, key, T, p);	//在左子树继续查找
	}else{
		return SearchBST(T->rchild, key, T, p);	//在右子树继续查找
	}
}

/*
当二叉排序树T中不存在关键字等于key的数据元素时
插入key并返回TRUE，否则返回FALSE
*/
bool InsertBST(BiTree *T, int key){
	BiTree p, s;
	if(!SearchBST(*T, key, NULL, &p)){
		//查找不成功
		s = (BiTree)malloc(sizeof(BiTNode));
		s->data = key;
		s->lchild = s->rchild = NULL;
		if(!p){
			*T = s;	//插入s为新的根节点
		}else if(key < p->data){
			p->lchild = s;	//插入s为左孩子
		}else{
			p->rchild = s;	//插入s为右孩子
		}
		return TRUE;
		}else{
			return FALSE;	//树种已有关键字相同的结点，不再插入
		}
}

/*********************构造二叉排序树************/
int i;
int a[10] = {62, 88, 58, 47, 35, 73, 51, 99, 37, 93};
BiTree T = NULL;
for(i = 0; i<10; i++){
	InsertBST(&T, a[i]);
}
/************************************************/

/*
若二叉排序树T中存在关键字等于key的数据元素时，则删除该数据元素结点，
并返回TRUE;否则返回FALSE
*/
bool DeleteBST(BiTree *T, int key){
	if(!T){
		return FALSE; 
	}else{
		if(key == T->data){
			//找到关键字等于key的数据元素
			return Delete(T);
		}else if(key < T -> data){
			return DeleteBST(T -> lchild, key);
		}else{
			return DeleteBST(T -> rchild, key);
		}
	}
}

```

* **哈希表（散列表）**

> ​	散列表是根据关键字而直接进行访问的数据结构。也就是说，散列表建立了关键字和存储地址之间的一种直接映射关系。
>
> ​	这种对应关系称为散列函数，又称为哈希(Hash) 函数。按这个思想，采用散列技术将记录存储在一块连续的存储空间中，这块连续存储空间称为散列表或哈希表(Hash table)。那么关键字对应的记录存储位置我们称为散列地址。

* 散列函数的构造方法

1. 直接定址法

​				直接取关键字的某个线性函数值为散列地址，散列函数为：
$$
H(key)=key或H(key)=a∗key+b
$$


2. 数字分析法

> 分析数字关键字在各位上的变化情况，取比较随机的位作为散列地址。

3. 平方取中法

> 平方取中法比较适合于不知道关键字的分布，而位数又不是很大的情况。
>
> 字面意思：平方之后取中间的数字作为散列地址

4. 除留余数法

​				散列函数：

> h（key） = key % 17
>
> > * p取素数（避免余数相关性）

5. 随机数法

$$
H(key)=random(key)
$$



* 处理散列冲突

​				开放地址法（闭散列表）和链地址法（开散列表法）

1. 线性探测法

> 从冲突的的下一个位置起，依次寻找空的散列地址
>
> > * 存在堆积现象

​	公式：
$$
H 
i
​
 (key)=(f(key)+d 
i
​
 )%m (d 
i
​
 =1,2,3,...,m−1)
$$
​	此时： di = 0,1,2,3,...k

2. 二次（平方）探测法

​		公式
$$
H 
i
​
 (key)=(f(key)+d 
i
​
 )%m (d 
i
​
 =1,2,3,...,m−1)
$$
​	此时：di = 0,1,-1,2^2,-2^2,....k^2,-k^2

3. 拉链法

> 将所有关键字为同义词的记录存储在一个单链表中，我们称这种表为同义词子表，在散列表中只存储所有同义词子表的头指针。



















