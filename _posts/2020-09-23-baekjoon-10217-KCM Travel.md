---
title: "[Baekjoon,10217] KCM Travel"
date: 2020-09-23 15:20:00 -0400
categories: algorithm 
tags:
- baekjoon 
- BFS
- DP
use_math: true
---
이 문제의 그래프는 음수 가중치를 가진 간선이 없는 그래프이다. 
그리고 단일 시작점에서 도착 점 까지의 최단거리를 구해야 하므로 다익스트라의 최단거리를 구하는 알고리즘을 이용하면 된다.  

---

# 다익스트라 알고리즘 이용 
## 비용에 제한이 있다. 
이 문제는 일반적인 문제와 다르게 가중치 말고도 비용의 제한이 추가 되었다. 
이 때문에 각 정점까지의 최단 거리를 저장하는 배열 $dist$를 $2$차원으로 선언하여 각 정점 까지의 최단거리를 현재 사용한 비용 별로 저장하여야 한다. 
이렇게 해야 
예를 들어 아래와 같은 입력이 주어졌을 때를 보자. 
```
// input
1
4 6 4
1 2 2 1
2 3 2 1
3 4 3 1
1 3 2 5
// output
// 6
```
이는 아래와 같은 가중치(비용, 소요 시간)를 가진 방향 그래프가 형성 된다.  
![graph](/assets/images/graph.png)  
이 그래프의 최단 거리는 비용의 제한이 $6$이므로 $ 1-3-4 $ 순으로 방문한 $6$이다. 

그런데 만약 $dist$배열을 $1$차원으로 생성하였다면 $4$번 정점을 방문하지 못한다. 
그 이유는 다음과 같다. 
- 먼저 그래프에서 $1$번 정점을 방문하고 우선순위 큐에 $2,3$번 정점을 추가한다.  
  이 다음 가장 거리가 가까운 $2$번 정점을 방문하고 $dist[2]$를 갱신한다.  
  그 후 $3$번 정점을 방문하는데 현재까지 소요된 비용은 $4$이며 $dist[3]$을 $2$로 갱신한다. 
  이 다음 $3$번 정점과 $4$번 정점을 잇는 간선의 가중치를 보면 비용이 $3$이므로 방문할 수 없다.  

- 다시 큐에 들어있던 $3$번 정점을 꺼내어 방문하려는데 이미 $dist[3]$에는 $2$가 저장되어 있어 방문하지 못한다.  

따라서 $dist$배열을 각 정점에 대한 거리를 현재 사용한 비용별로, 즉 아래와 같이 2차원으로 구현해야 한다. 
- $dist[vertex][cost]$ 

## 가지치기 
$dist[vertex][cost]$가 newDuration으로 새로 갱신된 시점에서 $dist[duration][cost+n], \ \ (0<n<limit-cost)$ 인 $dist$ 값 들이 
newDuration보다 크다면 이 값들이 의미가 없다. 왜냐하면 비용을 더 들여서 이동했는데 소요시간이 더 크기 때문이다. 
따라서 이 값들을 전부 newDuration으로 초기화 해주면, 다음에 이 정점을 방문하는 의미 없이 큰 소요시간을 가진 정점은 걸러질 것이다.  
이때 비용을 많이 들인 경로는 소요시간이 더 작을 테니 값들을 하나 씩 초기화 하다가 newDuration보다 작은 값이 발견되면 반복문을 탈출해야 더욱 빠르게 구현할 수 있다. 

## 구현 
```cpp
#include <iostream>
#include <vector>
#include <cstring>
#include <queue>
using namespace std;

const int INF=987654321;
const int MAX_N=100;
const int MAX_M=10000;

int N,limit;
// < <소요 시간, 소요 비용>, 연결된 공항 > 
vector<pair<pair<int,int>,int> > adj[MAX_N]; 

int getMinTime(int start, int target)
{
    // dist[정점][소요비용]
    vector<vector<int> > dist(N,vector<int>(limit+1,INF));
    // pq<소요 시간(음수), 소요 비용(음수)>, 공항> >
    priority_queue<pair<pair<int,int>,int> > pq;
    dist[start][0]=0;
    pq.push({ {0, 0}, start});
    while(!pq.empty())
    {
        int here=pq.top().second;
        int hereDuration=-pq.top().first.first;
        int hereCost=-pq.top().first.second;
        pq.pop();
        // 이미 더 빠른 경로를 알고있다면
        if(dist[here][hereCost]<hereDuration)
            continue;
        // 목표에 도착
        if(here==target)
            return hereDuration;
        
        for(int i=0;i<adj[here].size();++i)
        {
            int there=adj[here][i].second;
            int nextDuration=hereDuration+adj[here][i].first.first;
            int nextCost=hereCost+adj[here][i].first.second;
            // 더 짧은 경로를 발견 했으며, 비용 한도를 넘지 않는 경우 
            if(dist[there][nextCost]>nextDuration && nextCost<=limit)
            {
                // 가지치기 
                for(int cost=nextCost+1;cost<=limit;++cost)
                {
                    if(dist[there][cost]<=nextDuration)
                        break;
                    dist[there][cost]=nextDuration;
                }
                dist[there][nextCost]=nextDuration;
                pq.push({{-nextDuration,-nextCost},there});
            }
        }
    }
    return -1;
}

int main()
{
    int testCase;
    cin>>testCase;
    while(testCase--)
    {
        int k;
        cin>>N>>limit>>k;
        for(int i=0;i<MAX_N;++i)
            adj[i].clear();
        for(int i=0;i<k;++i)
        {
            int u,v,c,d;
            scanf("%d %d %d %d",&u,&v,&c,&d);
            --u;--v;
            adj[u].push_back({{d,c},v});
        }
        int ret=getMinTime(0,N-1);
        if(ret!=-1)
            cout<<ret<<endl;
        else
            cout<<"Poor KCM"<<endl;
    }

    return 0;
}
```

---
# 참고
- 시나모온, [백준 - 10217 KCM Travel](https://sina-sina.tistory.com/101), 2020, 
```
