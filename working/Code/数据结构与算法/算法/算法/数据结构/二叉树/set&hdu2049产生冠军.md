```c++
/*1.产生冠军的的条件： 获胜的人没输过一次，最终获胜的人只有一个
  2.获胜的人和失败的人分别存储
  3.若获胜的人在失败的组中出现，则舍去，若最终剩余人数唯一，则为冠军*/
#include<iostream>
#include<set>
#include<string>
using namespace std;
set<string> win;
set<string> los;
int main()
{
	int n;
	while(scanf_s("%d",&n)!=EOF){
		if (!n)break;
		win.clear();//格式化
		los.clear();
		while (n--) {
			string winner, loser;
			cin >> winner>>loser;
			win.insert(winner);//插入
			los.insert(loser);
		}
		int leap = 0;
		for (set<string>::iterator it = win.begin(); it != win.end(); it++) {
			if (los.find(*it) == los.end()) {//find： 返回元素值为elem的第一个元素，如果没有返回end()
				++leap;
			}
		}
		if (leap == 1)printf("Yes\n");
		else printf("No\n");
	}
	return 0;
}