---
title: "[Programmers]블록 게임"
date: 2020-09-13 22:10:00 -0400
categories: algorithm 
tags:
- programmers
- brute_force
---
# 해결 
여러가지 조건들을 유념하며 구현하면 되는 문제이며 간단한 아이디어로 구현이 깔끔해진다. 
그 아이디어는 만들어 질 수 있는 크기 내에서 모든 칸을 계산하는 것이다. 

## 직사각형 구역 안을 탐색
모든 조각은 2x3 or 3x2 직사각형에서 빈 칸을 뚫으면 만들 수 있다. 
맵에 모든 곳을 이 직사각형 구역 안을 탐색하는데 위쪽 블록부터 제거하기 위해 왼쪽 위에서 부터 탐색을 한다. 

이때 직사각형 구역에 빈 칸이 3개 이상이면 조각이 아니기 때문에 탐색을 그만둔다. 
또한 빈 곳이 있는데 위쪽이 막혀 그 곳에 1x1조각을 떨어뜨릴 수 없을 경우와 
구역 내에 다른 조각이 섞여 있다면 탐색을 그만한다. 

조각이 완성되면 해당 조각을 지워준다. 
더 이상 완성시킬 조각이 없다면 결과를 반환하면 된다. 
# 구현 
구현 시 탐색할 직사각형 구역이 범위에 벗어나지 않도록 주의해야 한다. 
```cpp
#include <string>
#include <vector>
using namespace std;

vector<vector<int> > map;
int n;

bool canDrop(int r, int c) {
    for(int i=0;i<r;++i)
        // 막힌 부분이 존재하면 안된다.
        if(map[i][c]!=0) 
            return false;
    return true;
}

bool canFill(int row, int col, int h, int w) {
    int empty=0, id=-1;
    for(int y=row;y<row+h;++y)
        for(int x=col;x<col+w;++x) {
            // 블록이 빈 곳을 찾았다.
            if(map[y][x]==0) {
                // 빈 곳인데 떨굴 수 없다.
                if(!canDrop(y,x)) 
                    return false;
                // 빈 공간이 3칸이 넘으면 없는 블록이다.
                if(++empty>2)
                    return false;
            }
            else {
                // 블록에 다른 블록이 섞여있다.
                if(id!=-1 && id!=map[y][x])
                    return false;
                id=map[y][x];
            }
        }
    for(int y=row;y<row+h;++y)
        for(int x=col;x<col+w;++x)
            map[y][x]=0;
    return true;
}

int solution(vector<vector<int>> board) {
    map=board;
    n=map.size();
    int ret=0;
    bool can=true;
    while(can) {
        can=false;
        // 모든 맵의 3x2 or 2x3 블럭을 찾는다.
        for(int i=0;i<n;++i)
            for(int j=0;j<n;++j) {
                // 블럭이 범위를 벗어나면 안된다.
                if(i<=n-3 && j<=n-2 && canFill(i,j,3,2)) {
                    ++ret;
                    can=true;
                }
                else if(i<=n-2 && j<=n-3 && canFill(i,j,2,3)) {
                    ++ret;
                    can=true;
                }
            }
    } 
    return ret;
}
```
