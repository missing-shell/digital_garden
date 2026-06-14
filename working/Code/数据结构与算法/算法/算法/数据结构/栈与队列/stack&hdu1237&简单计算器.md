```c++
#include <iostream>
#include <cstdio>
#include <string>
#include <stack>
using namespace std;
int main()
{
    char  c;
    double a, b;
    while (scanf_s("%lf", &a) != EOF)
    {
        stack <double> num;
        c = getchar();
        if (c == '\n' && a == 0)
            break;
        num.push(a);
        scanf_s("%c", &c);
        while (scanf_s("%lf", &b))
        {
            if (c == '*')
            {
                a = num.top();
                num.pop();
                num.push(a * b);
            }
            else if (c == '/')
            {
                a = num.top();
                num.pop();
                num.push(a / b);
            }
            else if (c == '+')
                num.push(b);
            else if (c == '-')
                num.push(-b);
            if (getchar() == '\n')
                break;
            scanf_s("%c", &c);
            getchar();
        }
        double ans = 0.0;
        while (!num.empty())
        {
            ans += num.top();
            num.pop();
        }
        printf("%.2lf\n", ans);
    }
    return 0;
}
```
[[STL模板#^61b99a]]

