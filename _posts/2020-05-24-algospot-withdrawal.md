---
title: "[Algospot]WITHDRAWAL"
date: 2020-05-24 17:11:00 -0400
categories: algorithm 
tags:
- algospot
- bisection
toc: false
---
# 해결
처음에는 r/c가 큰 것부터 철회하면 해결 할 수 있을 줄 알았으나 이 방법으로는 답을 구할 수 없다.  

## 질문 바꾸기 
**"적절히 과목을 철회했을 때 얻을 수 있는 최소 누적 등수"** 라는 질문을 **"적절히 과목을 철회해 누적 등수 x 이하가 될 수 있나?"** 라고 변경하면 접근이 수월해 진다.
하지만 sum(r[i]/c[i])<=x 을 구하기에는 어떤 과목들을 골라야 할지 알 수 없다.
여기서 적절히 수식을 조작하면 아래와 같은 식이 나온다.  
- 0<=sum(x*c[i]-r[i]) 
- v[i]=x*c[i]-r[i]

즉, 누적 등수가 x일때 x와 전체등수를 곱한 값이 현재등수와 일치하면 x가 정확하다고 할 수 있다. 
여기서 v를 정렬하고 가장 큰 k개의 원소를 더해서 쉽게 풀 수 있다.  

## 탐욕법 
v에서 k개 이상의 집합을 모두 선택해 보는 것이 아니라 가장 큰 원소 k개만 선택한다.  
```cpp
#include <iostream>
#include <algorithm>
#include <vector>
using namespace std;

int n,k;
int c[1000],r[1000];

bool decision(double average)
{
    //sum(r[i]/c[i])<=x
    //sum(r[i])<=x*sum(c[i])
    //0<=sum(x*c[i]-r[i])
    //v[i]=x*c[i]-r[i]를 계산한다.
    vector<double> v;
    for(int i=0;i<n;++i)
        v.push_back(average*c[i]-r[i]);
    sort(v.begin(),v.end());
    
    //greedy
    double sum=0;
    for(int i=n-k;i<n;++i)
        sum+=v[i];
    return sum>=0;
}

double optimize()
{
    double lo=-1e-9, hi=1;
    for(int it=0;it<100;++it)
    {
        double mid=(lo+hi)/2.0;
        if(decision(mid))
            hi=mid;
        else
            lo=mid;
    }
    return hi;
}

int main()
{
    int testCase;
    cin>>testCase;
    while(testCase--)
    {
        cin>>n>>k;
        for(int i=0;i<n;++i)
            cin>>r[i]>>c[i];
        
        printf("%.10lf \n",optimize());
    }

    return 0;
}

```
