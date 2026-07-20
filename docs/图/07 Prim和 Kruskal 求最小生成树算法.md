# C++ Prim和 Kruskal 求最小生成树算法



生成树：在图中找一棵包含图中的所有节点的树，生成树是含有图中所有顶点的无环连通子图。所有可能的生成树中，权重和最小的那棵生成树就叫最小生成树。在无向加权图中计算最小生成树，使用最小生成树算法的现实场景中，图的边权重一般代表成本、距离这样的标量。

## `kruskal`

这里就用到了贪心思路：将所有边按照权重从小到大排序，从权重最小的边开始遍历，如果这条边和`mst`中的其它边不会形成环，则这条边是最小生成树的一部分，将它加入`mst`集合；否则，这条边不是最小生成树的一部分，不要把它加入`mst`集合。

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

class UF {
  private:
   //连通分量的个数
   int count;
   //数组:每一个节点对应一个集合（树）
   int parent[100];
 public:
   //构造函数
   UF(int n) {
     this->count=n;//0  9
     for(int i=0; i<n; i++) {
      this->parent[i]=i;
     }
   }
  /*
  * 合并函数，构建边
  */
  void union_(int a,int b) {
   //检查a 和 b 是不是同一个集合
   //查找彼此的根节点
   int aroot= this->findRoot(a);
   int broot=this->findRoot(b);
   //如果是 什么都不做
   if(aroot==broot)return ;
   //如果不是，合并
   //  this->parent[broot]=aroot;
   this->parent[aroot]=broot;
   this->count--;
  }
  /*
  * 查找某一个节点的父节点
  */
  int findRoot(int id) {
   //根节点特点，parent[id]==id
   while( this->parent[id]!=id ) {
    id= this->parent[id];
   }
   return id;
  }
  /*
  *检查连通性
  */
  bool isConnection(int a,int b) {
   int aroot= this->findRoot(a);
   int broot=this->findRoot(b);
   return aroot==broot;
  }
};

struct Edge {
 int f;
 int t;
 int w;
//重写
 bool operator<( const Edge & edge  )  const {
  return this->w< edge.w;
 }
};

int main() {
 int n=3,m=3; // 
 int weight=0;
 cin>>n>>m;
 UF uf(n+1); //  
 Edge edges[100];
 int f,t,w;
 for(int i=0; i<m; i++) {
  cin>>f>>t>>w;
  edges[i]= {f,t,w};
 }
 //排序
 sort( edges,edges+m);
 for(int i=0; i<m; i++) {
   if( uf.isConnection( edges[i].f,edges[i].t  ) ==true  ) {
     continue;
   }
   weight+=edges[i].w;
   uf.union_(edges[i].f,edges[i].t );
 }
 cout<< weight;
 return 0;
}
```

## Prim

在图中进行广度搜索，使用优先队列存储边的信息。

```cpp
#include <bits/stdc++.h>

using namespace std;
/*
* 顶点类型
*/
struct Ver {
 //顶点编号
 int vid=0;
 //第一个邻接点
 int head=0;
 //是否被选择
 int isSel=0;
};

/*
* 边
*/
struct Edge {
 //邻接点
 int to;
 //下一个
 int next=0;
 //权重
 int weight;
 //是否已经添加
 bool isVis=0;
 //重载函数
 bool operator<( const Edge & edge ) const {
  return this->weight > edge.weight;
 }
};

class Prim {
 private:
  //存储所有顶点
  Ver vers[100];
  //存储所有边
  Edge edges[100];
  //顶点数，边数
  int v,e;
  //优先队列
  priority_queue<Edge> proQue;
  //最小生成树的权重
  int weight=0;

 public:
  Prim( int v,int e ) {
       this->v=v;
       this->e=e;
       init();
  }
  void init() {
      
   for(int i=1; i<=v; i++) {
    //重置顶点信息
    vers[i].vid=i;
    vers[i].head=0;
    vers[i].isSel=0;
   }

   int f,t,w;
   for(int i=1; i<=e; i++) {
        cin>>f>>t>>w;
        //设置边的信息
        edges[i].to=t;
        edges[i].weight=w;
        //头部插入
        edges[i].next=vers[f].head;
        vers[f].head=i;
   }
  }

  void pushQue(Ver & ver) {

       for( int i=ver.head; i!=0; i=edges[i].next) {
        int to=edges[i].to;
        if(  edges[i].isVis==false ) {
         edges[i].isVis=true;
         //入队列
         proQue.push( edges[i] );
        }
       }
  }

  void prim(int start) {


   //初始化优先队列
   vers[start].isSel=true;
   pushQue(vers[start]);

   while( !proQue.empty() ) {
    //出队列
    Edge edge=proQue.top();
    proQue.pop();

    if(  vers[edge.to].isSel  ==true)continue;
    vers[edge.to].isSel  =true;

    weight+=edge.weight;
    //邻接边入队
    pushQue( vers[edge.to] );
   }

   for(int i=1; i<=v; i++) {
    if( vers[i].isSel )cout<<i<<"\t";
   }
   cout<<weight<<endl;

  }

};

int main() {
 int v,e;
 cin>>v>>e;
 Prim prim(v,e);
 prim.prim(1);
 return 0;
}
```


