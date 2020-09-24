---
title: "[codeforces] Dijkstra?"
date: 2020-09-24 23:00:00 -0400
categories: algorithm 
tags:
- codeforces 
- BFS 
- dijkstra
use_math: true
toc: false 

---

영어로 된 문제와 친해지기 위해 codeforces의 문제를 풀어봤다. 
1900의 Difficulty를 가진 문제이며 다익스트라의 최단 경로 알고리즘을 이용해 실제 최단 경로를 출력하면 된다. 

구현한 코드를 아래에서 볼 수 있다. 경로가 없는 경우 빈 벡터를 반환하였고, 실제 최단 경로를 $parent[]$ 배열을 이용하여 생성했다. 
주의할 점은 정점의 최대 수가 $10^5$이고 간선의 가중치의 최대 값이 $10^6$이기 때문에 $long long$형을 사용해야 한다. 
```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <climits>
using namespace std;

const int MAX_V=1000001;

int V,E;
vector<pair<int,int> > adj[MAX_V];

vector<int> dijkstra()
{
    priority_queue<pair<long long,int> > pq;
    vector<long long> dist(V+1,LLONG_MAX);
    vector<int> parent(V+1,-1);
    pq.push(make_pair(0,1));
    dist[1]=0;
    parent[1]=1;
    while(!pq.empty())
    {
        int here=pq.top().second;
        long long cost=-pq.top().first;
        pq.pop();
        if(dist[here]<cost)
            continue;
        // 목적지에 도달한 경우 
        if(here==V)
        {
            // 경로 생성 
            vector<int> route;
            for(int p=V;p!=1;p=parent[p])
                route.push_back(p);
            route.push_back(1);
            return route;
        }
        for(int i=0;i<adj[here].size();++i)
        {
            int there=adj[here][i].first;
            long long nextDist=adj[here][i].second+cost;
            if(dist[there]>nextDist)
            {
                dist[there]=nextDist;
                parent[there]=here;
                pq.push(make_pair(-nextDist,there));
            }
        }
    }
    return vector<int>();
}

int main()
{
    cin>>V>>E;
    for(int i=0;i<E;++i)
    {
        int a,b,c;
        cin>>a>>b>>c;
        adj[a].push_back(make_pair(b,c));
        adj[b].push_back(make_pair(a,c));
    }
    
    vector<int> ret=dijkstra();
    if(ret.empty())
        cout<<-1<<endl;
    else
        for(int i=ret.size()-1;i>=0;--i)
            cout<<ret[i]<<' ';

    return 0;
}

```
