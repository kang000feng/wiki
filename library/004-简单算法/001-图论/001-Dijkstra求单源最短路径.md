# Dijkstra求单源最短路径问题
## 代码实现
```c++
#include <iostream>
#include <cstdio>
#include <cstring>
#include <algorithm>
#include <math.h>
#include <map>
using namespace std;
const int maxn = 505;
const int MAX_INT = 0x7FFFFFFF;

int n,m,c1,c2;
int maps[maxn][maxn];
int num[maxn],a[maxn],b[maxn];
int dis[maxn],vis[maxn];
int t1,t2;

void dijkstra(){

	//将起点放入最短路集合S
	vis[c1] = 1;
	b[c1] = num[c1];
	//初始化与起点相连的点的距起点距离
	for(int i=0;i<n;i++){
		dis[i] = maps[c1][i];
		if(maps[c1][i] != MAX_INT && i!=c1)
			b[i] = b[c1] + num[i];
	}
	//双层循环，外层循环是需要白重复N次，直到找到所有的点
	for(int i=0;i<n;i++){
	    //先找到不在S集合中的距离起点值最小的点，并将其加入最小集合S
		int mins = MAX_INT,minc = 0;
		for(int j=0;j<n;j++)
			if(dis[j]!=MAX_INT && vis[j]==0)
				if(dis[j]<mins){
					mins = dis[j];
					minc = j;
				}
		vis[minc] = 1;
		//更新与这个点想连的所有点的最短距离
		for(int j=0;j<n;j++){
			if(vis[j] == 0 && maps[minc][j] + dis[minc] < dis[j] && maps[minc][j] + dis[minc] >0 ){
				dis[j] = maps[j][minc] + dis[minc];
				b[j] = b[minc] + num[j];
				a[j] = a[minc];
			}
			//此题的特殊处理，一般Dijkstra没有此处理步骤
			else if(vis[j] == 0 && maps[minc][j] + dis[minc] == dis[j]){
				a[j] = a[j]+a[minc];
				if(b[j] < num[j] + b[minc])
					b[j] = num[j] + b[minc];
			}
		}
	}
}

int main(){
    //输入数据
	cin>>n>>m>>c1>>c2;
	for(int i=0;i<maxn;i++)
		for(int j=0;j<maxn;j++)
			maps[i][j] = MAX_INT;

	//初始化数据
	for(int i=0;i<n;i++){
		dis[i] = 0;
		a[i] = 0;
		b[i] = 0;
		vis[i] = 0;
		a[i] = 1;
	}
	for(int i=0;i<n;i++)
		cin>>num[i];
	for(int i=0;i<m;i++){
		cin>>t1>>t2;
		cin>>maps[t1][t2];
		maps[t2][t1] = maps[t1][t2];
	}
	//核心算法
	dijkstra();
	//输出结果
	cout<<a[c2]<<" "<<b[c2];
	return 0;
}
```

## Dijkstra算法核心思路：

1. 起点默认加入最短路径集合S（可用标记数组区分），使用dis[maxn]数组存储初始所有点与起点的距离，从单源起点出发，更新与其相连的所有点的距离dis[maxn].
2. 从dis[maxn]选出不在S集合中最小的点，加入S集合表示已经是最短路径了，然后从这个点出发，遍历其所有相连的点，更新与其相连的所有点的dis[maxn]值（即此点距离+两点距离与原本点距离比较，小于则进行更新。），之后重复此步骤，直至所有点都在集合S内（除非存在孤立点）。
3. 输出最短距离即可，若打印路径则可用数组记录某个点的前驱结点。
