--
title: "[Algospot]ALLERGY"
date: 2020-09-06 10:20:00 -0400
categories: algorithm 
tags :
- algospot 
- DFS 
- SCC
- 2-SAT
---

# 해결
## 무작정 시도
현재 차례의 사람이 이기면 1을 반환하고 아니면 0을 반환하는 식으로 코드를 구현하려 했다. 
하나라도 이기는 경우가 있으면 이기는 것, 즉 이기는 경우가 없으면 지는것이다. 

기본값을 0로 두었다가 반복문에서 이기는 경우가 안나오면 지도록 설정했다.
그리고 비트 마스크 없이 배열로 맵을 표현한뒤 해결하려 했다.
한 칸씩 움직이며 현재 위치가 비었을때 블록을 배치하려 했으나 코드가 너무 복잡해져서 포기하고 비트 마스크를 이용하기로 했다.

## 비트마스크 이용

비트 마스크를 이용하면 반복문을 이용하여 한칸씩 비교할 필요가 없어진다.
또한 맵 전체에 대한 블록들을 만들어 놓고 현재 맵과 & 연산자를 이용하여 배치 가능한지 쉽게 파악할 수 있다.

실제 구현시 아래와 같은 방법을 이용했다.    
- 맵 전체에 대해 놓을 수 있는 블록들을 비트로 표현하여 moves에 저장해 놓는다.
- L자 블록을 만들때 2x2 셀에서 한칸씩 제거하며 블록들을 저장한다.

```c++
for(int y=0;y<4;++y)
    for(int x=0;x<4;++x)
    {
        vector<int> cells;
        for(int dy=0;dy<2;++dy)
            for(int dx=0;dx<2;++dx)
                cells.push_back(cell(y+dy,x+dx));
        //2X2 꽉차있는 셀에서 하나씩 빼며 만든다.
        int square=cells[0]+cells[1]+cells[2]+cells[3];
        for(int i=0;i<4;++i)
            moves.push_back(square-cells[i]);
    }
```
## 승리조건
- 현재 차례에 둘 곳이 더이상 없으면 패배한다.
- 상대방이 지면 내가 승리한다.
- 현재 차례에 신의한수가 존재하면 이긴다.

즉, 현재 하나의 경우라도 승리하는 경우가 있다면 승리하는 것이다.

## 구현 
코드는 아래와 같다.  
```c++
#include <iostream>
#include <cstring>
#include <vector>
using namespace std;

vector<int> moves;
int n;
char cache[1<<25];

// 셀의 위치(y,x)를 비트로 변환(y*5+x) 해준다.
inline int cell(int y,int x)
{
    return 1<<(y*5+x);
}

// 블록이 있으면 비트가 켜진다.
void init(int& board)
{
    for(int y=0;y<5;++y)
        for(int x=0;x<5;++x)
        {
            char tmp; cin>>tmp;
            if(tmp=='#')
                board|=cell(y,x);
        }
    memset(cache,-1,sizeof(cache));
}

void preCalc(void)
{
    //4칸 블록 계산
    for(int y=0;y<4;++y)
        for(int x=0;x<4;++x)
        {
            vector<int> cells;
            for(int dy=0;dy<2;++dy)
                for(int dx=0;dx<2;++dx)
                    cells.push_back(cell(y+dy,x+dx));
            //2X2 꽉차있는 셀에서 하나씩 빼며 만든다.
            int square=cells[0]+cells[1]+cells[2]+cells[3];
            for(int i=0;i<4;++i)
                moves.push_back(square-cells[i]);
        }
    //2칸 블록 계산
    for(int i=0;i<5;++i)
        for(int j=0;j<4;++j)
        {
            moves.push_back(cell(i,j)+cell(i,j+1));
            moves.push_back(cell(j,i)+cell(j+1,i));
        }
}

char play(int board)
{
    char& ret=cache[board];
    if(ret!=-1)
        return ret;
    
    ret=0;
    for(int i=0;i<moves.size();++i)
        // 배치가능할때
        if((board & moves[i]) == 0)
            // 상대가 이길 수 없으면 내가 이긴 것이다.
            if(!play(board | moves[i]))
            {
                ret=1;
                break;
            }
    
    return ret;
}

int main(void)
{
    int testCase; cin>>testCase;
    
    while(testCase--)
    {
        int board=0;
        
        init(board);
        preCalc();
        
        if(play(board))
            cout<<"WINNING"<<endl;
        else
            cout<<"LOSING"<<endl;
    }
    
    return 0;
}

```
# 회고
교재에 있는 코드는 정말 예술적이다. 미리 블록들을 배치시켜 놓는다는 발상은 코드를 완전 뒤바꿨다.
