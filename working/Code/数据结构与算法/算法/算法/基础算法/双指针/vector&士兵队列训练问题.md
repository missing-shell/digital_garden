```c++
#include<iostream> 
#include<vector>
using namespace std;
vector<int> ans;
int main()
{
	int N;
	scanf_s("%d", &N);
	while (N--) {
		ans.clear();//更新
		int n; 
		scanf_s("%d", &n);
		for (int i = 1; i <= n; i++)ans.push_back(i);
		while (ans.size() > 3) {//人数小于等于三直接输出
			for (int i = 1; i <ans.size(); i++) {
				ans.erase(ans.begin() + i);//删除报数为二的
			}
			if (ans.size() <= 3)  break;//只要一轮报数完毕人数小于等于三就结束
			for(int i=2;i<ans.size(); i + 2) {
				ans.erase(ans.begin() + i);
			}
		}
		for (int i = 0; i < ans.size(); i++) {
			printf("%d%c", ans[i], i == ans.size() - 1 ? '\n' : ' ');
		}
	}
	return 0;
}