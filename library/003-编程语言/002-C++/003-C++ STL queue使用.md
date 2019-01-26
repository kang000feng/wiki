# C++ STL queue使用

## queue常用方法
实现BFS

### 普通队列

```c++
#include <iostream>
#include <string>
#include <cstdio>
#include <cstring>
#include <algorithm>
#include <math.h>
//头文件
#include <queue>
#include <map>
using namespace std;

#define MAXN 105

typedef struct TreeNode{
	string id;
	int x;
	int lv;
	string son[MAXN];
}TreeNode;

TreeNode root;
int n,m;
//初始化
queue<TreeNode> q;
map<string,TreeNode> maps;
int level = 0,t = 0;

void bfs(TreeNode node){
    //向队列里面插入元素
	q.push(node);
	while(!q.empty()){
	    //返回队首元素，但是此元素并没有出队
		node = q.front();
		//将队首元素出队，但是没有返回队首元素
		q.pop();
		if(node.lv != level){
			level = node.lv;
			cout<<t<<" ";
			t = 0;
		}
		if(node.x > 0){
			for(int i=0;i<node.x;i++){
				map<string,TreeNode>::iterator it = maps.find(node.son[i]);
				TreeNode temp;
				if(it == maps.end()){
					temp.id = node.son[i];
					temp.x = 0;
				}
				else
					temp = it->second;
				temp.lv = node.lv + 1;
				q.push(temp);
			}
		}
		else
			t++;
	}
	cout<<t;
}

int main(){
	cin>>n>>m;
	for(int i=0;i<m;i++){
		TreeNode node;
		cin>>node.id>>node.x;
		for(int j=0;j<node.x;j++){
			cin>>node.son[j];
		}
		maps.insert(pair<string,TreeNode>(node.id,node));
		if(node.id == "01"){
			root = node;
			root.lv = 0;
		}
	}
	bfs(root);
	return 0;
}
```


### 优先队列


```c++
#include <iostream>
#include <algorithm>
#include <cstdio>
#include <string.h>
#include <cstring>
#include <math.h>
#include <queue>

using namespace std;

priority_queue<int> Q1;

priority_queue<int,vector<int>,greater<int> > Q2;

struct T
{
	int x,y;
};

struct cmp
{
	bool operator ()(const T &a,const T &b){
		if(a.x == b.x)
			return a.y > b.y;
		return a.x < b.x;
	}
};

priority_queue<T,vector<T>,cmp> Q3;

string n,t;
int d;

int main(){
	Q1.push(5);
	Q2.push(5);

	Q1.push(3);
	Q2.push(3);

	Q1.push(7);
	Q2.push(7);

	Q1.push(9);
	Q2.push(9);

	Q1.push(12);
	Q2.push(12);

	while(!Q1.empty()){
		cout<<Q1.top()<<" "<<Q2.top()<<endl;
		Q1.pop();
		Q2.pop();
	}

	T w;
	w.x = 1;
	w.y = 3;
	Q3.push(w);

	w.x = 5;
	w.y = 1;
	Q3.push(w);

	w.x = 4;
	w.y = 4;
	Q3.push(w);

	w.x = 6;
	w.y = -1;
	Q3.push(w);

	while(!Q3.empty()){
		w = Q3.top();
		Q3.pop();
		cout<<w.x<<" "<<w.y<<endl;

	}


}
```
