---
title: "[Programmers]단어 퍼즐"
date: 2020-09-13 10:20:00 -0400
categories: algorithm 
tags:
- programmers
- DP 
---
# 해결 
문제에서 유념할 조건은 아래와 같다.  
> - 단어 퍼즐을 중복하여 사용할 수 있다. 
> - 단어 조각 길이는 5 이하이다. 

## 삽질 
제일 첫 접근은 조각을 선택해 그 조각이 문자열에 있으면 해당 조각 부분을 대문자로 변환하여 모든 문자열이 대문자가 될 때까지 재귀하는 것이었다. 
하지만 이러한 접근은 굉장히 중복되는 부분이 많아 느렸다. 당연히 시간초과를 받았다.  

다음 접근은 BFS를 이용해보는 것이었다. 이 또한 `banana` 에서 `na`를 선택하면 `baNAna`가 되는 것 처럼 대문자로 변환하며 bfs를 시도했다. 
하지만 변환하는 과정이 복잡하기 때문에 마찬가지로 시간초과를 받았다.

## 최적해를 찾자 
문제를 가장 작게 나누어 최적해를 찾아 반복되는 부분을 저장하여 해결하는 동적 계획법을 이용하면 해결할 수 있다. 
완성되어야 하는 목표에서 조각들을 선택해 줄여 나가며, 글자가 완성된 경우 값을 반환하여 최소 값인지 비교한다. 
이때 문자열을 매개변수로 넘기지 않고 인덱스를 넘기면 메모이제이션을 할 수 있다.

# 구현 
이러한 함수를 이용하여 해결하면 아래와 같다. 
구현 시 문자열이 중복되어 선택될 수 있으니 `begin`에서부터 한 글자씩 더하며 strs에 있는 지 찾았다.  
이때 strs에 문자열이 있는지 찾는 계산을 `set<string>`을 만들어 빠르게 하였다. 
글자가 존재한다면 다음 인덱스로 넘어가는 식으로 구현했다. 
```cpp
#include <iostream>
#include <string>
#include <vector>
#include <set>
#include <algorithm>
#include <cstring>
using namespace std;

const int INF=987654321;

int cache[20001];
// 목표 문자 길이
int n;
// 목표 문자
string T;
// 단어 조각들 
set<string> s;

// 완성본 글자가 [begin,n-1)인 경우 몇 개의 단어를 선택해야 하는 지 반환 
int dp(int begin) {
    // 기저 사례: index가 넘어간 경우
    if(begin>=n)
        return 0;
    // 메모이제이션
    int& ret=cache[begin];
    if(ret!=-1)
        return ret;
    
    ret=INF;
    // 빈 문자열 부터 begin에서 시작하고 end에서 끝나는 글자들을 만들어 본다.
    string next="";
    for(int end=begin;end<n;++end) {
        next+=T[end];
        // 글자가 strs에 있는 경우
        if(s.count(next))
            ret=min(ret,dp(end+1)+1);
        // strs는 길이가 5글자 이하다.
        if(begin<end-5)
            break;
    }
    return ret;
}

int solution(vector<string> strs, string t) {
	memset(cache,-1,sizeof(cache));
	T=t;
    n=t.size();
    // 빠른 탐색을 위해 set 사용
	for(int i=0;i<strs.size();++i)
	    s.insert(strs[i]);
	    
	int ret=dp(0);
	return ret==INF ? -1 : ret;
}
```
