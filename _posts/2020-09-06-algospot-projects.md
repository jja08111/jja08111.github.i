---
title: "[Algospot]PROJECTS"
date: 2020-09-06 10:20:00 -0400
categories: 
- algorithm 
tags:
- algospot 
- networkflow
---
# 주의 
이 문제는 채점 사이트에 등록되지 않았기 때문에 저의 코드는 틀린 코드일 수 있습니다.  
아래에 있는 예제만 통과한 코드입니다.

# 예제 
교재에 있는 예제 데이터는 아래와 같다.  
```c++
2
2 2
10 10
5 10
1 0
1 1
5 5
260 60 140 350 500
250 100 150 300 100
1 0 0 0 0
1 1 1 0 0
0 0 1 1 0
0 0 0 1 0
0 0 0 1 1
```

# 해결 

### 그래프 모델링 
먼저 주어진 데이터들을 그래프로 표현해야 한다.  
각 사업을 진행하기 위해서는 필요한 장비들이 있다.  
하지만 **장비를 이미 만들어 놓았다면 다시 만들 필요는 없다.**  

따라서 국책 사업을 표현하는 정점의 집합 P와 각 장비를 표현하는 정점의 집합 E 그리고 소스와 싱크로 구성한다고 했을 때, 
먼저 소스에서 국책 사업으로 수익만큼 용량을 갖는 간선으로 표현한다.  
그리고 국책 사업에서 필요한 장비에 간선을 연결하되 그 용량을 무한대로 설정하고, 장비에서 싱크로 가격만큼 용량을 설정한다.  

이렇게 하면 **사업에서 장비로 이미 유량이 흐른 경우 장비에서는 싱크로 가격만큼만 용량이 설정되어 있어 이미 구매한 장비를 표현 가능**하다.  

### 컷을 이용한 최대 순이익 구하기 
그렇다면 이러한 그래프로 어떻게 최대 순이익을 구한다는 것인가?  
방법은 최소 컷에 있다.  

먼저 네트워크의 정점들을 컷 S, T로 나눈다. S에 포함된 장비는 모두 구입하고, S에 포함된 사업들에 전부 응찰하는 것이다.  
교재에 있는 그림 32.11을 보면 순이익은 전체 P의 이익의 합에서 최소 컷의 용량을 빼면 구할 수 있는 것을 볼 수 있다.  

이에 대한 증명 과정은 아래와 같다.  

용량이 유한한 컷이 주어졌을 때 컷의 용량(C)은 얼마인가?  
- 싱크와 (P에서 S를 제외한 사업)을 연결한 간선, 즉 응찰하지 않을 사업들의 이익의 합
- S에 포함된 장비의 가격, 즉 응찰하기로 한 사업에서 필요한 장비들의 가격의 합 

위 두 항을 더하면 컷의 용량이 된다.  
그런데 **위 두 항은 모두 순이익을 최대화 하기 위해서 최소화 해야 하는 값**이다.  

실제로 순이익을 표현하는 식과 위 식을 더하면 알 수 있다.  
- 순이익(X) : (S에 포함된 사업들(P)의 이익) - (S에 포함된 장비의 가격)

두 식을 더하면 아래와 같다.  
- X + C  
= (P에서 S를 제외한 사업) + (S에 포함된 장비의 가격) + (S에 포함된 사업들(P)의 이익) - (S에 포함된 장비의 가격)  
= (P에서 S를 제외한 사업) + (S에 포함된 사업들(P)의 이익)  
= 전체 사업(P)의 수익의 합

즉, X = (전체 사업(P)의 수익의 합) - C 인 X를 구하기 위해서는 **전체 사업의 수익은 변하지 않으니 최소인 C를 찾는 것**이다.  
최소인 C는 최소 컷의 용량인데 **"최소 컷 최대 유량 정리"에 따라 최소 컷의 용량은 네트워크의 최대유량과 같기 때문에** 이를 구하면 된다.  
```c++
#include <iostream>
#include <vector>
#include <queue>
#include <cstring>
#include <numeric>
using namespace std;

const int MAX_NM=101;
const int MAX_V=100+100+10;
const int INF=987654321;

int V,n,m;
int profit[MAX_NM], cost[MAX_NM], requires[MAX_NM][MAX_NM];
int capacity[MAX_V][MAX_V], flow[MAX_V][MAX_V];

int networkFlow(int source, int sink)
{
    int ret=0;
    while(true)
    {
        // 너비 우선 탐색으로 증가 경로를 찾는다.
        vector<int> parent(V,-1);
        queue<int> q;
        parent[source]=source;
        q.push(source);
        while(!q.empty() && parent[sink]==-1)
        {
            int here=q.front(); q.pop();
            for(int there=0;there<V;++there)
                if(parent[there]==-1 && capacity[here][there]-flow[here][there]>0)
                {
                    q.push(there);
                    parent[there]=here;
                }
        }
        // 증가 경로가 없으면 종료한다.
        if(parent[sink]==-1) break;
        // 증가 경로를 통해 유량을 얼마나 보낼지 결정한다.
        int amount=INF;
        for(int p=sink;p!=source;p=parent[p])
            amount=min(amount,capacity[parent[p]][p]-flow[parent[p]][p]);
        // 증가 경로를 통해 유량을 보낸다.
        for(int p=sink;p!=source;p=parent[p])
        {
            flow[parent[p]][p]+=amount;
            flow[p][parent[p]]-=amount;
        }
        ret+=amount;
    }
    return ret;
}

int maxProfit()
{
    // 네트워크를 만든다.
    const int SRC=0, SNK=1;
    V=n+m+2;
    memset(capacity,0,sizeof(capacity));
    memset(flow,0,sizeof(flow));
    for(int i=0;i<n;++i)
        capacity[SRC][2+i]=profit[i];
    for(int i=0;i<m;++i)
        capacity[2+n+i][SNK]=cost[i];
    for(int i=0;i<n;++i)
        for(int j=0;j<m;++j)
            if(requires[i][j])
                capacity[2+i][2+n+j]=INF;
    // 모든 사업의 수익의 합을 구한다.  
    int M=accumulate(profit,profit+n,0);
    int C=networkFlow(SRC,SNK);
    return M-C;
}

int main()
{
    int testCase;
    cin>>testCase;
    while(testCase--)
    {
        cin>>n>>m;
        for(int i=0;i<n;++i)
            cin>>profit[i];
        for(int i=0;i<m;++i)
            cin>>cost[i];
        for(int i=0;i<n;++i)
            for(int j=0;j<m;++j)
                cin>>requires[i][j];
        
        cout<<maxProfit()<<endl;
    }
    return 0;
}
```
