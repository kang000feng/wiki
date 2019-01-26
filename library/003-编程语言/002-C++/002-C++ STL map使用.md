# C++ STL map使用

## 示例
```c++
#include <iostream>
#include <map>
using namespace std;

void map_traversal(map<int,int> m1){
	//map 遍历
	map<int,int>::iterator it;
	for(it=m1.begin();it!=m1.end();it++)
		cout<<"Key: "<<it->first<<" ,Value: "<<it->second<<endl;
}


//map基本的初始化、增删改查等操作
void map_basic(){
	//map初始化，分别对应键值类型
	map<int,int> m1;

	map<int,string> m2;

	//map插入方式1，使用pair键值对方式插入
	m1.insert(pair<int,int>(1,2));
	//map插入方式2，使用数组赋值形式
	m1[2] = 5;
	map_traversal(m1);

	//map查找
	map<int,int>::iterator it;
	//find 方法返回iterator对象，如果没有找到返回的是Map.end()
	it = m1.find(1);
	cout<<"Find result:"<<endl;
	if(it!=m1.end()){
		cout<<"Key: "<<it->first<<" ,Value: "<<it->second<<endl;
	}
	//count 方法返回key值个数，只可能是0或1
	cout<<m1.count(1)<<endl;

	//erase 移除key及其对应的value
	m1.erase(1);
}

struct compare{
	bool operator()(const int& x1,const int& x2){
		return x1 > x2;
	}
};

void map_sort(){
	//map 排序，默认会按key从小到大排序加入compare会按照compare规则排序
	map<int,double,compare> ms;
	ms[2] = 1.0;
	ms[4] = 2.0;
	//iterator指针也需要加上排序规则
	map<int,double,compare>::iterator it;
	//加排序规则 直接使用 count会报错，find不会

}

map<int,double,compares> s;

int main(){

	map_basic();

	return 0;
}
```
