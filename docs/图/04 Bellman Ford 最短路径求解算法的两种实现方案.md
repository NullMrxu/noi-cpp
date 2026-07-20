# C++ Bellman Ford 最短路径求解算法的两种实现方案

## 概念

贝尔曼-福特算法取自于创始人理查德.贝尔曼和莱斯特.福特，本文简称 BF 算法。BF 算法属于迭代、穷举算法，算法效率较低，如果图结构中顶点数量为 n，边数为 m ，则该算法的时间复杂度为 m*n ，还是挺大的。

核心思想

1、更新顶点的权重：计算任一条边上一端顶点（始点）到另一个端顶点（终点）的权重。新权重=顶点（始点）权重+边的权重，然后使用新权重值更新终点的原来权重值。

2、更新原则：只有当顶点原来的权重大于新权重时，才更新。

## 不使用队列

```cpp
#include <stdio.h>
#include <iostream>
int main()
{

    int dis[10],bak[10],i,k,n,m,u[10],v[10],w[10],check,flag;
    int inf=99999;
    //读入n和m，n表示顶点个数，m表示边的条数
    scanf("%d %d",&n,&m);
    //读入边
    for(i=1; i<=m; i++)
        scanf("%d %d %d",&u[i],&v[i],&w[i]);
    //初始化dis数组，这里是1号顶点到其余各个顶点的初始路程
    for(i=1; i<=n; i++)
        dis[i]=inf;

    dis[1]=0;

    //Bellman-Ford算法核心语句
    for(k=1; k<=n-1; k++)
    {
        //将dis数组备份至bak数组中
        for(i=1; i<=n; i++)bak[i]=dis[i];

        //进行轮松弛
        for(i=1; i<=m; i++)

            if(dis[ v[i] ] > dis[ u[i] ] +w[i]  )
                dis[ v[i]  ]=dis[ u[i] ]+w[i];

        check=0;

        for(i=1; i<=n; i++)
            if( bak[i]!=dis[i])
            {
                check=1;
                break;
            }
        //如果dis数组没有更新，提前退出循环结束算法
        if(check==0)break;
    }
    //检测负权回路
    flag=0;

    for(i=1; i<=m; i++)
        if( dis[v[i]]> dis[u[i]]+w[i]) flag=1;

    if(flag==1)printf("此图含有负权回路");
    else
    {
        //输出最终的结果
        for(i=1; i<=n; i++)
            printf("%d\t",dis[i]);

    }
    return 0;
}
//测试用例
5 5
2 3 2
1 2 -3
1 5 5
4 5 2
3 4 3
//输出
0   -3   -1   2    4
```

## Bellman-Ford的队列优化

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
 //起点到此顶点的距离（顶点权重）,初始为 0 或者无穷大
 int dis=0;
 //是否入队
 bool isVis=false;

 //重载函数
 bool operator<( const Ver & ver ) const {
  return this->dis<ver.dis;
 }

 void desc() {
  cout<<vid<<" "<<dis<<endl;
 }
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
};

class Graph {
 private:
  const int INF=99999;
  //存储所有顶点
  Ver vers[100];
  //存储所有边
  Edge edges[100];
  //顶点数，边数
  int v,e;
  //起点到其它顶点之间的最短距离
  int dis[100];
  //优先队列
  queue<int> que;

 public:
  Graph( int v,int e ) {
   this->v=v;
   this->e=e;
   init();
  }
  void init() {
   for(int i=1; i<=v; i++) {
    //重置顶点信息
    vers[i].vid=i;
    vers[i].dis=INF;
    vers[i].head=0;
    vers[i].isVis=false;
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

   for(int i=1; i<=v; i++) {
    dis[i]=vers[i].dis;
   }
  }

  void spfa(int start) {
   //初始化队列,起点到起点的距离为 0
   vers[start].dis=0;
   dis[start]=0;
   vers[start].isVis=true;

   que.push(start);

   while( !que.empty() ) {
    //出队列
    Ver ver=vers[  que.front()  ];
    ver.desc();
    que.pop();

    //找到邻接顶点 i 是边集合索引号
    for( int i=ver.head; i!=0; i=edges[i].next) {
     int v=edges[i].to;
     //更新距离
     if( vers[ v ].dis > ver.dis + edges[i].weight ) {
      vers[ v ].dis = ver.dis+edges[i].weight;
      dis[ v ]= vers[ v ].dis;
     }
     //入队列 有可能更新了队列中没有更新
     if( vers[ v ].isVis==false ) {
      vers[ v ].isVis=true;
      que.push( v );
     }
    }
   }
  }

  void show() {
   for(int i=1; i<=v; i++) {
    cout<<dis[i]<<"\t";
   }
  }

};

int main() {
 int v,e;
 cin>>v>>e;
 Graph graph(v,e);
 graph.spfa(1);
 graph.show();
 return 0;
}
//测试用例
5 7
1 2 2
1 5 10
2 3 3
2 5 7
3 4 4
4 5 5
5 3 6
//输出
0  2 5  9  9
```

