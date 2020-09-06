---
title: "[Algospot]MEETINGROOM"
date: 2020-09-06 10:20:00 -0400
categories: algorithm 
tags :
- algospot 
- DFS 
- SCC
- 2-SAT
comments : true
---

# 해결

이 문제는 각 팀이 주간회의 월간회의 중 어느 것을 선택할지를 결정해야 한다.

회의번호를 0부터 2n-1까지 부여하여 i번 회의를 할 것인지 여부를 불린 값 변수 A(i)로 나타낸다.

1.  t번 팀의 회의는 2t와 2t+1 이니 
    **모든 t번 팀에 대해 (A(2t) || A(2t+1))이 참**이어야 한다.

2.  두 회의가 겹친다면 최소한 둘 중 하나는 개최되면 안되거나 하나도 열리지 말아야 한다.
    따라서 **(!A(a) || !A(b)) 이 참**이어야 한다.


위 사항을 모두 && 연산자로 묶으면 논리식을 얻을 수 있다.  
이는 논리곱 정규형(conjuctive normalform)으로 표현된다.  

이때 논리식을 논리곱 정규형으로 표현하였을 경우 각 절에 최대 두개의 변수만이 존재하는 경우 이 문제를 2-SAT 문제라고 부른다.
2-SAT 문제는 다른 SAT 문제와 달리 그래프를 이용해 다항 시간에 해결 가능하다.  



## 변수 함의 그래프 생성

2-SAT 문제에서 논리식의 각 절은 두 개의 변수로 구성되어 있다는 점을 이용한다.

**절을 구성하는 두 변수 중 하나는 반드시 참**이어야 한다.  
**절이 (A0 || A1) 일때 1번회의가 거짓이면 0번 회의는 참**이다. 이를 'P이면 Q이다' 형태의 관계로 표현할 수 있다.  

이것은 아래와 같이 필요충분조건으로 표현이 가능하다.  

-   0번 회의가 개최되지 않는다(!A(0)) -> 1번 회의가 개최된다.(A(1))
-   1번 회의가 개최되지 않는다(!A(1)) -> 0번 회의가 개최된다.(A(0))

위 내용을 변수 A(i)에 대해 각각 **A(i)와 !A(i)를 표현하는 두개의 정점을 포함하는 방향 그래프를 만들고**, 각 절마다 위의 표기에 따라 방향 간선을 추가하면 함의 그래프를 만들 수 있다.  
함의 그래프(imlication graph)는 논리식에 포함된 변수들의 값에 대한 요구 조건을 표현한 그래프를 말한다.  

구현시 주의할 점은 책에서 함의 그래프를 형성할 때 (X || Y) 절에 대해 둘 다 참인 경우를 고려하지 않은 것이다. 사이트에서는 이를 고려해야 한다.  
```c++
vector<vector<int> > adj;
// 두 구간 a와 b가 서로 겹치지 않는지를 확인한다.
bool disjoint(const pair<int,int>& a, const pair<int,int>& b)
{
    return a.second<=b.first || b.second<=a.first;
}

// meetings[]가 각 팀이 하고 싶어하는 회의 시간의 목록일 때,
// 이를 2-SAT 문제로 변환한 뒤 함의 그래프를 생성한다.
// i번 팀은 meetings[2*i] 혹은 meetings[2*i+1] 중 하나 시간에 회의실을 써야 한다.
void makeGraph(const vector<pair<int,int> >& meetings)
{
    int vars=meetings.size();
    // 그래프는 각 변수마다 두 개의 정점을 갖는다.
    adj.clear(); adj.resize(vars*2);
    for(int i=0;i<vars;i+=2)
    {
        // 각 팀은 i번 회의나 j번 회의 둘 중 하나는 해야 한다.
        // (i or j) 절을 추가한다.
        int j=i+1;
        adj[i*2+1].push_back(j*2); // ~i -> j
        adj[j*2+1].push_back(i*2); // ~j -> i
        adj[i*2].push_back(j*2+1); // i -> ~j
        adj[j*2].push_back(i*2+1); // j -> ~i
    }
    for(int i=0;i<vars;++i)
        for(int j=0;j<i;++j)
        {
            // i번 회의와 j번 회의가 서로 겹칠 경우
            if(!disjoint(meetings[i],meetings[j]))
            {
                // i번 회의가 열리지 않거나, j번 회의가 열리지 않아야 한다.
                // (~i or ~j) 절을 추가한다.
                adj[i*2].push_back(j*2+1); // i -> ~j
                adj[j*2].push_back(i*2+1); // j -> ~i
            }
        }
}
```

## 함의 그래프를 이용해서 2-SAT 문제 해결

논리식의 어떤 절 (X||Y)가 거짓이라면 즉, 주간회의와 월간회의 둘 다 열리지 않아 IMPOSSIBLE인 경우는 어떻게 될까? 일단 두 변수 모두 거짓이어야 한다. 
그리고 이 절은 아래의 두 개의 간선으로 표현된다.

- !X -> Y
- !Y -> X

두 변수가 거짓이니 참에서 거짓으로 가는 것을 알 수 있다. (!A or !B) 또한 마찬가지이다.

따라서 **참 정점에서 거짓 정점으로 가는 경로는 없어야 한다.**
그래프가 **사이클인 경우 무조건 참 정점에서 거짓 정점으로 가는 경로가 생기기 때문에 사이클인 그래프는 논리식을 만족하는 변수 조합이 없다.**


## 강결합 요소 이용하기

앞서 이야기 했든이 답이 존재하지 않는 경우는 한 사이클에 참 정점과 거짓 정점이 모두 포함되는 때이다.  
따라서 한 사이클 내의 정점들을 한 묶음으로 처리하면 된다.  

즉, SCC를 만들어 처리하면 되는 것이다.  

이때 각 절을 두 개의 간선으로 표현했기 때문에 X와 !X는 1:1 대응 관계가 존재하게 된다.
이는 한 SCC가 참이라면 그의 반대 SCC는 항상 거짓이어야 함을 알 수 있다.

결과적으로 이제는 원래 그래프를 SCC별로 압축한 그래프를 또 하나의 함의 그래프로 보고 문제를 풀어도 된다.  

이 압축된 그래프는 SCC별로 압축했기 때문에 DAG이다.

DAG는 들어오는 간선이 하나도 없는 정점이 항상 존재한다. 
이 정점을 **거짓으로 분류**하여 이 정점에 대응되는 **반대 정점을 참으로 분류**한 뒤 그래프에서 이 정점을 지운다. 


## 구현 
들어오는 간선이 없는 정점을 선택해 반복적으로 삭제하는 대신, 압축된 그래프를 위상 정렬한 순서대로 처리하면 된다.  
이를 구현하기 위해 sccID를 first로 두어 역순으로 정렬하였고 vertex를 second로 취해 쉽게 접근하였다.

```c++
// adj에 함의 그래프의 인접 리스트 표현이 주어질 떄, 2-SAT 문제의 답을 반환한다.
vector<int> solve2SAT()
{
    int n=adj.size(); // 변수의 수
    // 함의 그래프의 정점들을 SCC 별로 묶는다.
    vector<int> label=tarjanSCC();
    // 이 SAT 문제를 푸는 것이 불가능한지 확인한다. 한 변수를 나타내는 두 정점이
    // 같은 강결합 요소에 속해있을 경우 답이 없다.
    for(int i=0;i<n;i+=2)
        if(label[i]==label[i+1])
            return vector<int>();
    
    // SAT 문제를 푸는 것이 가능하다. 답을 생성하자.
    vector<int> value(n,-1);
    // tarjan 알고리즘에서 SCC 번호는 위상정렬의 역순으로 배정된다.
    // SCC 번호의 역순으로 각 정점을 정렬하면 위상 정렬 순서가 된다.
    vector<pair<int,int> > order;
    for(int i=0;i<n;++i)
        order.push_back(make_pair(-label[i],i));
    sort(order.begin(), order.end());
    
    // 각 정점에 값을 배정한다.
    for(int i=0;i<n;++i)
    {
        int vertex=order[i].second;
        int variable=vertex/2, isTrue=(vertex%2==0);
        if(value[variable]!=-1)
            continue;
        // ~A가 A보다 먼저 나왔으면 A는 참
        // A가 ~A보다 먼저 나왔으면 A는 거짓 
        value[variable]=!isTrue;
    }
    return value;
}
```

# 완성본 
위 내용을 토대로 코드를 작성하면 아래와 같다.  
```c++
#include <iostream>
#include <vector>
#include <stack>
#include <algorithm>
using namespace std;

vector<vector<int> > adj;
vector<int> discovered, sccId;
stack<int> st;
int sccCounter, vertexCounter;

// 두 구간 a와 b가 서로 겹치지 않는지를 확인한다.
bool disjoint(const pair<int,int>& a, const pair<int,int>& b)
{
    return a.second<=b.first || b.second<=a.first;
}

// meetings[]가 각 팀이 하고 싶어하는 회의 시간의 목록일 때,
// 이를 2-SAT 문제로 변환한 뒤 함의 그래프를 생성한다.
// i번 팀은 meetings[2*i] 혹은 meetings[2*i+1] 중 하나 시간에 회의실을 써야 한다.
void makeGraph(const vector<pair<int,int> >& meetings)
{
    int vars=meetings.size();
    // 그래프는 각 변수마다 두 개의 정점을 갖는다.
    adj.clear(); adj.resize(vars*2);
    for(int i=0;i<vars;i+=2)
    {
        // 각 팀은 i번 회의나 j번 회의 둘 중 하나는 해야 한다.
        // (i or j) 절을 추가한다.
        int j=i+1;
        adj[i*2+1].push_back(j*2); // ~i -> j
        adj[j*2+1].push_back(i*2); // ~j -> i
        adj[i*2].push_back(j*2+1); // i -> ~j
        adj[j*2].push_back(i*2+1); // j -> ~i
    }
    for(int i=0;i<vars;++i)
        for(int j=0;j<i;++j)
        {
            // i번 회의와 j번 회의가 서로 겹칠 경우
            if(!disjoint(meetings[i],meetings[j]))
            {
                // i번 회의가 열리지 않거나, j번 회의가 열리지 않아야 한다.
                // (~i or ~j) 절을 추가한다.
                adj[i*2].push_back(j*2+1); // i -> ~j
                adj[j*2].push_back(i*2+1); // j -> ~i
            }
        }
}

int scc(int here)
{
    int ret=discovered[here]=vertexCounter++;
    st.push(here);
    for(int i=0;i<adj[here].size();++i)
    {
        int there=adj[here][i];
        if(discovered[there]==-1)
            ret=min(ret,scc(there));
        else if(sccId[there]==-1)
            ret=min(ret,discovered[there]);
    }
    if(ret==discovered[here])
    {
        while(true)
        {
            int t=st.top();
            st.pop();
            sccId[t]=sccCounter;
            if(t==here) break;
        }
        sccCounter++;
    }
    return ret;
}

vector<int> tarjanSCC()
{
    discovered=sccId=vector<int>(adj.size(),-1);
    sccCounter=vertexCounter=0;
    for(int i=0;i<adj.size();++i)
        if(discovered[i]==-1)
            scc(i);
    
    return sccId;
}

// adj에 함의 그래프의 인접 리스트 표현이 주어질 떄, 2-SAT 문제의 답을 반환한다.
vector<int> solve2SAT()
{
    int n=adj.size(); // 변수의 수
    // 함의 그래프의 정점들을 SCC 별로 묶는다.
    vector<int> label=tarjanSCC();
    // 이 SAT 문제를 푸는 것이 불가능한지 확인한다. 한 변수를 나타내는 두 정점이
    // 같은 강결합 요소에 속해있을 경우 답이 없다.
    for(int i=0;i<n;i+=2)
        if(label[i]==label[i+1])
            return vector<int>();
    
    // SAT 문제를 푸는 것이 가능하다. 답을 생성하자.
    vector<int> value(n,-1);
    // tarjan 알고리즘에서 SCC 번호는 위상정렬의 역순으로 배정된다.
    // SCC 번호의 역순으로 각 정점을 정렬하면 위상 정렬 순서가 된다.
    vector<pair<int,int> > order;
    for(int i=0;i<n;++i)
        order.push_back(make_pair(-label[i],i));
    sort(order.begin(), order.end());
    
    // 각 정점에 값을 배정한다.
    for(int i=0;i<n;++i)
    {
        int vertex=order[i].second;
        int variable=vertex/2, isTrue=(vertex%2==0);
        if(value[variable]!=-1)
            continue;
        // ~A가 A보다 먼저 나왔으면 A는 참
        // A가 ~A보다 먼저 나왔으면 A는 거짓 
        value[variable]=!isTrue;
    }
    return value;
}

int main(void)
{
    int testCase;
    cin>>testCase;
    while(testCase--)
    {
        int n;
        cin>>n;
        
        vector<pair<int,int> > meetings(2*n);
        for(int i=0;i<n;++i)
        {
            int a,b,c,d;
            cin>>a>>b>>c>>d;
            meetings[2*i]=make_pair(a,b);
            meetings[2*i+1]=make_pair(c,d);
        }
        
        makeGraph(meetings);
        vector<int> ret=solve2SAT();
        
        if(ret.empty())
        {
            cout<<"IMPOSSIBLE"<<endl;
            continue;
        }
        cout<<"POSSIBLE"<<endl;
        for(int i=0;i<n*2;++i)
            if(ret[i])
                cout<<meetings[i].first<<" "<<meetings[i].second<<endl;
    }
    return 0;
}
```
