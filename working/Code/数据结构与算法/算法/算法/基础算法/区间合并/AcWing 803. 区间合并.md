```c++
#include<iostream>
#include<algorithm>
#include<vector>

using namespace std;

typedef pair<int,int>PII;

void merge(vector<PII>& sege)
{
    vector<PII> res;
    sort(sege.begin(), sege.end());//按左端点排序

    int st = -2e9, ed = -2e9;//
    for (auto item : sege)
    {
        if (ed < item.first) {//情况1：两个区间无法合并
            if (st != -2e9) {
                res.push_back({ st,ed });//区间1放进res数组
            }
            st = item.first, ed = item.second;//维护区间2

        }
        else {//情况2：两个区间可以合并 


            ed = max(ed, item.second);
        }
    }
    if (st != -2e9)res.push_back({ st,ed });//保存剩余区间（最后一个序列，所以不可能继续进行合并）
    sege = res;
}

int main()
{
    int n;
    cin >> n;
    vector<PII> sege;
    for (int i = 0; i < n; i++)
    {
        int a, b;
        cin >> a >> b;
        sege.push_back({ a,b });
    }
    merge(sege);
    cout << sege.size() << endl;
    return 0;
}
```