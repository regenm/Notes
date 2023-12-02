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

