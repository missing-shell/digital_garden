每个医生都会有病人，每个病人有对应的给他治病的医生，并且有优先级，
每个医生先给优先级高的病人治病，如果优先级相同就按照进来的时间顺序进行排队。
```c++
#include<iostream>
#include<vector>
#include<algorithm>
#include<string>
#include<queue>
using namespace std;
const int N = 2010;
struct Node {
	int rank;
	int id;
	bool operator <(const Node& t2)const {
		if (rank != t2.rank) return rank < t2.rank;//优先队列排序
		return id > t2.id;
	}
};
int main()
{
	int n;
	while (scanf("%d", &n) != EOF) {
		priority_queue<Node>doc[4];//每个医生一个优先队列
		int a, id = 1;
		while (n--) {
			string s;
			cin >> s >> a;
			Node tmp;
			if (s == "IN") {
				scanf("%d", &tmp.rank );
				tmp.id = id; 
				id++;
				doc[a].push(tmp);//记录病人情况
			}
			else if(s=="OUT") {
				if (!doc[a].empty()) {
					printf("%d\n", doc[a].top().id);
					doc[a].pop();
				}
				else printf("EMPTY\n");
				 
			}
		}
	}
	return 0;
}
```
[[STL模板#^2e6b21]]