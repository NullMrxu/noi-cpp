# C++ Dijkstra 最短路径求解算法的两种实现方案



迪杰斯特拉算法`(Diikstra)` 是由荷兰计算机科学家狄克斯特拉于1959 年提出的，因此又叫狄克斯特拉算法。

核心思想，搜索到某一个顶点后，更新与其相邻顶点的权重。顶点权重的数据含义表示从起始点到此点的最短路径长度（也就是经过的所有边的权重之和）。DJ 算法搜索时，每次选择的下一个顶点是所有权重值最小的顶点，其思想是保证每一次选择的顶点和当前顶点权重都是最短的。所以，DJ是基于贪心思想。

## 矩阵存储

常规时间复杂度：`O(n)`，可以使用堆优化优先队列，时间复杂度降低到`O(logN)`。缺点是对于稀疏图而言空间浪费严重。

```cpp
#include <bits/stdc++.h>
using namespace std;

//矩阵，存储图
int graph[100][100];
//顶点、边数
int v,e;
//优先队列，使用数组
int pri[100];
//存储起点到其它顶点之间的最短距离
int dis[100];
//设置无穷大常量
int const INF =INT_MAX;

/*
*初始化函数
*/
void init()
{
    //初始化图中顶点之间的关系
    for(int i=1; i<=v; i++)
    {
        for(int j=1; j<=v; j++)
        {
            if( i==j )
            {
                //自己和自己的关系（权重）为 0
                graph[i][j]=0;
            }
            else
            {
                //任意两点间的距离为无穷大
                graph[i][j]=INF;
            }
        }
    }

    //交互式确定图中顶点之间的关系
    int f,t,w;
    for( int i=1; i<=e; i++ )
    {
        cin>>f>>t>>w;
        graph[f][t]=w;

    }

    //初始设编号为 1 的顶点为起始点,根据顶点的关系初始化起点到其它顶点之间的距离
    for(int i=1; i<=v; i++)
    {
        dis[i]=graph[1][i];
    }

    //初始化优先队列（也称为候选队列）
    for(int i=1; i<=v; i++ )
    {
        if(i==1)
        {
            //起始顶点默认为已经候选
            pri[i]=1;
            continue;
        }
        //其它顶点都可候选
        pri[i]=0;
    }

}

/*
*
*Dijkstra算法
*/
void dijkstra()
{
    for(int i=1; i<=v; i++)
    {

        //从候选队列中选择一个顶点，要求到起始顶点的距离为最近的
        int u=-1;
        int mi=INF;
        for( int j=1; j<=v; j++ )
        {
            if(pri[j]==0 && dis[j]<mi)
            {
                mi=dis[j];
                u=j;
            }
        }

        
        if(u!=-1)
            //找到后设置为已经候选
            pri[u]=1;
        else 
            //找不到就结束
            break;

        //查找与此候选顶点相邻的顶点，且更新邻接点与起点之间的距离
       //相当于在此顶点基础上向后延长
        for( int j=1; j<=v; j++ )
        {

            if(  graph[u][j]!=INF )
            {
                //找到相邻顶点
                if(dis[j]>dis[u]+graph[u][j]  )
                {
                    //更新
                    dis[j]=dis[u]+graph[u][j];
                }
            }
        }

    }

}

/*
*
*显示最后的结果
*/
void show()
{
    for(int i=1; i<=v; i++)
    {
        cout<<dis[i]<<"\t";
    }
}


int main()
{
    cin>>v>>e;
    init();
    dijkstra();
    show();

    return 0;
}
//测试用例
6  9
1 2 1
1 3 12
2 3 9
2 4 3
3 5 5
4 3 4
4 5 13
4 6 15
5 6 4
//输出
0  1  8  4  13  17
```

## 邻接表

整个时间复杂度可以优化到`O(M+N)logN`。在最坏的情况下`M(边数)`就是`N（顶点数）`，这样的话`(M+M)logN`要比`N`还要大。但是大多数情况下并不会有那么多边，因此`(M+M)logN`要比`N`小很多。

```cpp
#include <bits/stdc++.h>

using namespace std;
/*
* 顶点类型
*/
struct Ver
{
    //顶点编号
    int vid=0;
    //第一个邻接点
    int head=0;
    //起点到此顶点的距离（顶点权重）,初始为 0 或者无穷大
    int dis=0;
    //重载函数
    bool operator<( const Ver & ver ) const
    {
        return this->dis<ver.dis;
    }

    void desc(){
      cout<<vid<<" "<<dis<<endl;
    }
};

/*
* 边
*/
struct Edge
{
    //邻接点
    int to;
    //下一个
    int next=0;
    //权重
    int weight;
};

class Graph
{
private:
    const int INF=INT_MAX;
    //存储所有顶点
    Ver vers[100];
    //存储所有边
    Edge edges[100];
    //顶点数，边数
    int v,e;
    //起点到其它顶点之间的最短距离
    int dis[100];
    //优先队列
    priority_queue<Ver> proQue;

public:
    Graph( int v,int e )
    {
        this->v=v;
        this->e=e;
        init();
    }
    void init()
    {
       for(int i=1;i<=v;i++){
              //重置顶点信息
            vers[i].vid=i;
            vers[i].dis=INF;
            vers[i].head=0;
       }
        int f,t,w;

        for(int i=1; i<=e; i++)
        {
            cin>>f>>t>>w;
            //设置边的信息
            edges[i].to=t;
            edges[i].weight=w;
            //头部插入
            edges[i].next=vers[f].head;
            vers[f].head=i;
        }

        for(int i=1; i<=v; i++)
        {
            dis[i]=vers[i].dis;
        }
    }

    void dijkstra(int start)
    {
        //初始化优先队列,起点到起点的距离为 0
        vers[start].dis=0;
        dis[start]=0;

        proQue.push(vers[start]);

        while( !proQue.empty() )
        {
            //出队列
            Ver ver=proQue.top();
            ver.desc();
            proQue.pop();

            //找到邻接顶点 i 是边集合索引号
            for( int i=ver.head; i!=0; i=edges[i].next)
            {
                int v=edges[i].to;
    //更新距离
                if( vers[ v ].dis > ver.dis + edges[i].weight )
                {
                    vers[ v ].dis = ver.dis+edges[i].weight;
                    dis[ v ]= vers[ v ].dis;
                    //入队列
                    proQue.push( vers[v] );
                }

            }

        }
    }

    void show()
    {
        for(int i=1; i<=v; i++)
        {
            cout<<dis[i]<<"\t";
        }
    }

};

int main()
{
    int v,e;
    cin>>v>>e;
    Graph graph(v,e);
    int s;
    cin>>s;
    graph.dijkstra(s);
    graph.show();
    return 0;
}
```



