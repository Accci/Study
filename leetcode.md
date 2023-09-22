#### 1.两数之和(双指针)

```c++
#inlcude<vector>
#include <unordered_map>
using std::vector;
using std::unordered_map // 哈希
class Slotion{
    public:
    vector<int> twoSum(vector<int>nums, int target){
        unordered_map<int, int>record;
        for(int i= 0; i != nums.size(); ++i)
        {
            auto found = record.find(nums[i]); //返回的类型是迭代器
            if(found != record.end())
                return{found->second, i};
            record.emplace(target - nums[i], i);
        }
        return {-1, -1};
    }
};


//两指针
vector<int> twoSum(vector<int>& numbers, int target)
{
    int l = 0, r = numbers.size();
    while(l<r)
    {
        int sum = numbers[l] + numbers[r];
        if(sum == target)
            break;
        else if(sum < target)
            ++l;
        else
            --r;
    }
    return vector<int>{l+1, r+1}; //位置 等于 下标+1
}
```

#### 2.两数相加

```c++
#include <cstddef>
#include <cstdlib>
struct ListNode{
	int val;
	ListNode *next;
	ListNode():val(0),next(nullptr){}
	ListNode(int x):val(x),next(nullptr){}
	ListNode(int x, ListNode* next):val(x),next(next){}
};

class Solution{
    public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2){
        ListNode dummy(0), *tail = &dummy; //对象用 .  指针用 ->
        for(div_t sum{0, 0}; sum.quot || l1 || l2; tail = tail->next){
            if(l1){sum.quot += l1->val; l1 = l1->next;}
            if(l2){sum.quot += l2->val; l2 = l2->next;}
            sum = div(sum.quot, 10);
            tail->next = new ListNode(sum.rem);
        }
        return dummy.next;
    }
};

```

#### 3.无重复字符的最长子串

```c++
#include<string>
#include<unordered_map>
using std:: string;
using std::unordered_map;

class Solution{
    public:
    int lengthOfLongestSubstring(string s){
        int n = s.size();
        int ret = 0;
        unordered_map<char, int>count;
        for(int r = 0, l = 0; r < n; ++r){
            count[s[r]]++;
            while(count[s[r]] >= 2)
                count[s[l++]]--;
            ret = max(ret, r-l+1);
        }
        return ret;
    }
};
```

#### 4.寻找两个正序数组的中位数

```c++
//用割的思想+二分
#include <vector>
using std::vector;
#include <stdio.h>

class Solution{
    public:
    double findMedianSortedArrays(vector<int>& nums1, vector<int>&nums2){
        int n = nums1.size();
        int m = nums2.size();
        if(n > m)
            return findMedianSortedArrays(nums2, nums1);
        int Lmax1, Lmax2, Rmin1, Rmin2, c1,c2,l0 = 0, hi = 2*n;
        while(l0 <= hi)
        {
            c1 = (l0 + hi)/2;
            c2 = m + n - c1;
            
            Lmax1 = (c1 == 0)? INT_MIN : nums1[(c1 - 1) /2];
            Rmin1 = (c1 == 2*n)? INT_MAX : nums1[c1 / 2];
            Lmax2 = (c2 == 0)? INT_MIN : nums2[(c2 -1)/2];
            Rmin2 = (c2 == 2 * m)? INT_MAX : nums2[c2 /2];
            
            if(Lmax1 > Rmin2)
                hi = c1 - 1;
            else if(Lmax2 > Rmin1)
                l0 = c1 + 1;
            else 
                break;
        }
        return (max(Lmax1, Lmax2) + min(Rmin1, Rmin2))/2.0;
    }
};


//先合并在求中位数

class Solution{
    public:
    double findMedianSortedArrays(vector<int>& nums1, vector<int>&nums2){
        int n = nums1.size();
        int m = nums2.size();
        vector<int>ret(m+n, 0);
       	int i =0, j=0,k=0;
        while(i<n && j < m)
        {
            if(nums1[i] > nums2[j])
            	ret[k++] = nums2[j++];
            else if(nums1[i] < nums2[j])
                ret[k++] = nums1[i++];
            else
            {
                ret[k++] = nums1[i++];
            	ret[k++] = nums2[j++];
            }
        }
        while(i < n)
            ret[k++] = nums1[i++];
        while(j < m)
            ret[k++] = nums2[j++];
        if(k%2)
         	return ret[k/2]*1.0;
        else
            return (ret[k/2] + ret[(k+1)/2])/2.0;
    }
};
```

#### 5.最长回文子串

```c++
#include <string>
#include <vector>

using std::string;
class Solution{
    public:
    string longestPalindrome(string s)
    {
       int n = s.size();
       int maxlen = 0,st = -1.len = 0;
        vector<vector<bool>>f(n,vector<bool>(n, false));
        int i = 0, j = 0;
        for(;i < n; ++i)
        {
            f[i][i] = true;
            maxlen = 1;
            st = 0;
        }
        for(len = 2, len <= n; ++len)
        {
            for(i = 0; i < n; ++i)
            {
                j = len + i - 1;
                if(s[i] != s[j]) continue;
                if(len > 2 && !f[i+1][j-1])continue;
                f[i][j] = true;
                maxlen = len;
                st = i;
            }
        }
        return s.substr(st, st+maxlen);
    }
};

//另外一种方式
class Solution{
   public:
    string longestPalindrome(string s){
        int len = s.size();
        if(len == 0) return s;
        int start = 0, last = 0;
        for(int i = 0; i < len; ++i)
        {
            _longestPalindrome(s, i, i, start, last); // 奇数的情况
            _longestPalindrome(s, i, i+1, start, last); //偶数的情况 
        }
     	return s.substr(start, last-start+1);   
    }
    void _longestPalindrome(const string& s, int b, int e, int &start, int &last)
    {
        int len = s.size();
        while(b >= 0 && e < len && s[b] == s[e])
        {
            --b,++e;
        }
        ++b,--e;
        if(e - b > start - last)
        {
            start = b;
            last = e;
        }
    }
};
```

#### 6.N字形变换

```c++
class Solution{
    public:
    string convert(string s, int numRows)
    {
        vector<string>rows(numRows);
        int n = s.size();
        int flag = 1, idRow = 0;
        string res;
        if(n < numRows || numRows == 1)
            return s;
        for(int i = 0; i < n; ++i)
        {
            rows[idRow].push_back(s[i]);
            idRow += flag;
            if(idRow == numRows -1 || idRow == 0)
            {
                flag = -flag;
            }
        }
        for(auto row : rows)
            res += row;
        return res;
    }
};

//方法二
#include <string>
using std::string;
#include <vector>
using std::vector;
$include <numeric>
    
class Solution{
    public:
    string convert(string s, int nRows)
    {
        int m = 0, n = 0;
        if(s.empty() || nRows < 2) return s;
        vector<string>ret(nRows);
        for(size_t i = 0; i < s.size(); ++i){
            m = i % (nRows -1), n = i / (nRows -1);
            (n & 0x1 ? ret[nRows -m -1]:ret[m]) += s[i]; // 与 奇数 & 0x1 为真，偶数 & 0x1 为假
        }
        return std::accumulate(ret.cbegin(),ret.cend(),string());
    }
};
```

#### 7.整数反转

```c++
class Solution{
    public:
    int reverse(int x){
        long res = 0;
        do{
            res = res * 10 + x % 10;
        }while(x /= 10);
       	return (int)res == res? (int)res:0;
    }
};
```

#### 8.字符串转整数

```c++
#include <string>
#include <climits>

class Solution{
    public:
    int myAtoi(string s){
        int i = 0;
        int sign = 1;
        int result = 0;
        while(i < s.size() &&s[i] == ' '){
            i++;
        }
        if(i < s.size() &&(s[i] == '+' || s[i] == '-'))
            sign = (s[i++] == '+') ? 1:-1;
        
        while(i < s.size() &&isdigit(s[i])){
            int digit = s[i++] - '0';
            if(result > INT_MAX / 10 || (result == INT_MAX/10 && digit > 7))
                return (sign == 1)? INT_MAX : INT_MIN;
 			result = result * 10 + digit;
        }
        return result * sign;
    }
};
```

#### 9.回文数

```c++
//不用额外的空间
class Solution{
    public:
    bool isPalindrome(int x){
        if (x < 0) return false;
        long rev{0}, origin{x};
        do {
            rev = rev * 10 + x % 10;
        } while (x /= 10);
        return rev == origin;
    }
};
```

#### 10.正则表达式匹配

```c++
//自己看题解
class Solution {
public:
    bool isMatch(string s, string p) {
        int n = s.size();
        int m = p.size();
        vector<vector<bool>>dp(n+1, vector<bool>(m+1, false));
        dp[0][0] = true;
        for(int j = 1; j <= m; ++j)
        {
            if(p[j-1] == '*')
                dp[0][j] = dp[0][j-2];
        }
        for(int i = 1; i <= n;++i){
            for(int j = 1; j <= m; ++j){
                if(s[i-1] == p[j-1] || p[j-1] == '.')
                    dp[i][j] = dp[i-1][j-1];
                else if(p[j-1] =='*')
                {
                    if(s[i-1] != p[j-2] && p[j-2] !='.')
                        dp[i][j] = dp[i][j-2];
                    else
                        dp[i][j] = dp[i][j-1] ||dp[i][j-2] || dp[i-1][j];
                }
            }

        }
        return dp[n][m];
    }
};
```

#### 11 盛最多水的容器(双指针)

```c++
/*
该代码的基本思路是，使用双指针从两端开始向中间移动。我们维护一个变量res来记录当前能够盛放的最大水量。在每次迭代中，我们将左指针i指向的高度和右指针j指向的高度中较小的一个作为当前容器的高度，并将容器的宽度设置为指针之间的距离。然后，我们将res更新为当前容器能够盛放的最大水量。最后，我们将高度较小的指针向中间移动，并重复上述过程，直到两个指针相遇。

由于我们只需要遍历一次整个数组，所以该算法的时间复杂度为$O(n)$，其中n是数组的长度。空间复杂度为$O(1)$，因为我们只需要存储三个整数变量。
*/
int maxArea(vector<int>& height){
    int res = 0;
    int i = 0, j = height.size() -1;
    while(i < j){
        res = max(res, min(height[i], height[j])*(j-i));
        if(height[i] < height[j])
            ++i;
        else
            --j;
    }
    return res;
}
```



#### 12整数转罗马数

```c++
//从高位开始
string intToRoman(int num){
    string res="";
    vector<int>nums = {1000, 900, 500, 400, 100, 90, 50, 40, 10, 9, 5, 4, 1};
    vector<string>romans = {"M", "CM", "D", "CD", "C", "XC", "L", "XL", "X", "IX", "v", "IV", "I"};
    int i = 0;
    while(num > 0){
        int count = num/nums[i];
        while(count > 0){
            res += romans[i];
            --count;
            num -= nums[i];
        }
        ++i;
    }
    return res;
}
```

#### 13 罗马数字转整数

```c++
string intToRoman(int num){
    unordered_map(char,int) m = {{'I',1}, {'V', 5}, {'X', 10}, {'L',50}, {'C', 100},{'D', 500}, {'M',1000}};
    int res = 0;
    for(int i = 0; i < s.size(); ++i){
        if(i == s.size()-1 || m[s[i]] > s[m[s+1]])
            res += m[s[i]];
        else
            res -= m[s[i]];
    }
    return res;
}
```

#### 14 最长公共前缀

```c++
/*
在该代码中，我们首先对数组进行排序，然后比较第一个字符串和最后一个字符串的公共前缀，并将结果保存到结果字符串中。

由于我们只需要比较第一个字符串和最后一个字符串，所以该算法的时间复杂度为$O(mn\log n)$，其中m是字符串的平均长度，n是字符串的数量。空间复杂度为$O(1)$，因为我们只需要存储几个字符串变量。
*/
string longesCommonPrefix(vector<string>& strs){
    sort(strs.begin(), strs.end());
    if(strs.empty())
        return "";
    int n = strs.size(), i=0;
    string res;
    while(i < strs[0].size() && i < strs[n-1].size() && strs[0][i] == strs[n-1][i]){
        res += strs[0][i];
        ++i;
    }
    return res;
}
```



#### 15 三数之和

```c++
/*
该代码的基本思路是，首先对给定的数组进行排序。然后，我们从左到右遍历数组中的每个数字，将其作为三元组中的第一个数字。对于每个三元组，我们使用双指针从剩余的数组元素中选择另外两个数字，并计算它们的和。如果三个数字的和等于0，则将它们添加到结果数组中。如果三个数字的和小于0，则将左指针向右移动，以使和更大。如果三个数字的和大于0，则将右指针向左移动，以使和更小。在每次迭代中，我们需要注意避免重复的数字。

由于我们需要对整个数组进行排序，并且需要在每个三元组中使用双指针来搜索另外两个数字，所以该算法的时间复杂度为$O(n^2)$，其中n是数组的长度。空间复杂度为$O(1)$，因为我们只需要存储几个整数变量。
*/
vector<vector<int>> threeSum(vector<int>& nums){
    vector<vector<int>>res;
    int n = nums.size();
    for(int i = 0; i < n; ++i){
        if(i > 0 && nums[i] == nums[i + 1])
            continue;
        int j = i + 1, k = n - 1;
        while(j < k){
            int sum = nums[i] + nums[j] + nums[k];
            if(sum == 0){
                res.push_back({nums[i], nums[j], nums[k]});
                while(j < k && nums[j] == nums[j + 1]) ++j;
                while(j < k && nums[k] == nums[k - 1]) --k;
                ++j;
                --k;
            }else if(sum > 0)
                --k;
            else
                ++j;
        }
    }
    return res;
}
```



#### 16 最接近的三数之和

```c++
/*
该代码的基本思路是，首先对给定的数组进行排序。然后，我们从左到右遍历数组中的每个数字，将其作为三元组中的第一个数字。对于每个三元组，我们使用双指针从剩余的数组元素中选择另外两个数字，并计算它们的和。我们使用一个变量res来记录当前最接近目标值的三元组的和。在每次迭代中，如果当前三元组的和比res更接近目标值，则更新res。最后，我们返回res。

由于我们需要对整个数组进行排序，并且需要在每个三元组中使用双指针来搜索另外两个数字，所以该算法的时间复杂度为$O(n^2)$，其中n是数组的长度。空间复杂度为$O(1)$，因为我们只需要存储几个整数变量。
*/
int threeSumClosest(vector<int>&nums, int target){
    sort(nums.begin(), nums.end());
    int n = nums.size(), res = nums[0] + nums[1] + nums[2];
    for(int i = 0; i < n; ++i){
        int j = i + 1, k = n - 1;
        while(j < k){
            int sum = nums[i] + nums[j] + nums[k];
            if(abs(sum - target) < abs(res - target)){
                res = sum;
            }
            if(sum < target)
                ++j;
            else
                --k;
        }
    }
    return res;
}
```

#### 17 电话号码的组合

```c++
/*
在该代码中，我们使用深度优先搜索（DFS）的方法进行搜索，其中table存储了电话号码中每个数字对应的字符集合，例如table[2]表示数字2对应的字符集合为"abc"。我们从第一个数字开始，依次枚举该数字对应的每个字符，并将该字符添加到结果字符串中，然后递归处理下一个数字。当处理完所有数字后，将结果字符串添加到结果数组中。
*/
vector<string> letterCominations(string digits){
    vector<string> res;
    if(digits.empty())
        return res;
    vector<string>table={"","","abc","def","ghi","jkl","mno","pqrs","tuv","wxyz"};
    string path;
    dfs(res, path, table, digits, 0);
    return res;
}
void dfs(vector<string>&res, string& path, vector<string>&table, string&digits, int idx){
    if(idx == digits.size()){
        res.emplace_back(path);
        return;
    } 
    string str = table[digits[idx] - '0'];
    for(int i = 0; i < str.size(); ++i){
        path.push_back(str[i]);
        dfs(res, path, table, digits, idx+1);
        path.pop_back();
    }
}
```

#### 18 四数之和

```c++
/*
该解法首先对数组进行排序，然后使用两层循环遍历前两个数。在每次循环中，我们分别使用双指针法在剩余的部分中寻找和为特定值的两个数。在循环过程中，我们需要注意避免重复的结果。
*/
vector<vector<int>> fourSum(vector<int>& nums, int target) {
    vector<int64_t>nums64(nums.begin(), nums.end());
    vector<vector<int64_t>> result64 = fourSumHelper(nums64, static_cast<int64_t>(target));
    vector<vector<int>> result;
    for(const auto& res : result64){
        result.push_back(static_cast<int>(res.begin(), res.end()));
    }
    return result;
}

private:
	vector<vector<int64_t>> fourSUmHelper(vector<int64_t>& nums, int64_t target){
        vector<vector<int64_t>>result;
        if(nums.size() < 4){
            return result;
        }
        
        sort(nums.begin(), nums.end());
		int n = nums.size();        
        for(int i = 0 && i < n - 3; ++i){
            if(i > 0 && nums[i] == nums[i - 1]){
                continue;
            }
            if(nums[i] + nums[i + 1] + nums[ i + 2] + nums[i + 3] > target){
                break;
            }
            if(nums[i] + nums[n-3] + nums[n-2] + nums[n-1] < target){
                continue;
            }
            for(int j = 0; i < n -2; ++j){
                if(j > 0 && nums[j] == nums[j -1]) continue;
                if(nums[i] + nums[j] + nums[j+1] + nums[j + 2] > target){
                    break;
                }
                if(nums[i] + nums[j] + nums[n-2] + nums[n -1] < target){
                    continue;
                }
                int left = j + 1, right = n-1;
                while(left < right){
                    int64_t sum = nums[i] + numsp[j] + nums[left] + nums[right];
                    if(sum == target){
                        result.push_back({nums[i], nums[j], nums[left], nums[right]});
                        ++left;
                        --right;
                   	 	while(left < right && nums[left] == nums[left-1]) ++left;
                    	while(left < right && nums[right] == numd[right+1]) ++right;
                	}else if(sum < target){
                    	++left;
                	}else{
                    	--right;
                	}//else
                }//while
            }//for
        }//for
        return result;
    }
```



#### 19 删除链表的倒数第N个结点（快慢指针）

```c++

/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */

ListNode* removeNthFromEnd(ListNode* head, int n){
    ListNode* fast = head;
    ListNode* slow = head;
    while(n-- && fast){
        fast = fast->next;
    }
    if(!fast)
        return head->next;
    while(fast->next){
        fast = fast->next;
        slow = slow->next;
    }
    ListNode* tmp = slow->next;
    slow->next = slow->next->next;
    delete tmp;
    return head;
}
```

#### 20 有效括号

```c++
bool isValid(string s){
    stack<char>st;
    for(char c:s){
        if(c =='{' || c == '[' || c == '('){
            st.push(c);
        }else{
            if(st.empty())
                return false;
          	if(c == ')' && st.top() != '(')
                return false;
            if(c == ']' && st.top() != '[')
                return false;
            if(c == '}' && st.top() != '{')
                return false;
            st.pop();
        }
    }
    return st.empty();
}
```

#### 21 合并两个有序链表

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
ListNode* mergeTwoLists(ListNode* list1,ListNode* list2){
    ListNode *dummy = new ListNode(0);
    LisNode *p = list1, *q = list2, *cur = dummy;
    while(p && q){
        if(p->val < q->val){
            cur->next = p;
            p = p->next;
        }else{
            cur->next = q;
            q = q->next;
        }
        cur = cur->next;
    }
    if(p){
        cur = p->next;
    }
    if(q){
        cur = q->next;
    }
    return dummy->next;
}
```

#### 22 括号的生成

```c++
vector<string> generateParenthesis(int n){
    vector<string> res;
    stack<pair<string, pair<int, int>>>st;
    st.emplace(make_pair("(", make_pair(1,0)));
    while(!st.empty()){
        auto p = st.top();
        string s = p.first;
        st.pop()
        int open = p.second.first;
        int close = p.second.second;
        if(open == n && close ==n){
            res.emplace_back(s);
        }else{
            if(open < n)
                st.emplace(make_pair(s+"(",make_pair(open+1,close)));
            if(open > close)
                st.emplace(make_pair(s+")", make_pair(open, close+1)));
        }
    }
    return res;
}
```

#### 23 合并K个升序链表

```c++
//将使用优先队列来解决此问题
struct Compare{
    bool operato()(const ListNode *a, const ListNode *b){
        return a->val > b->val;
    }
};
ListNode* mergeKlists(vector<ListNode*>& lists){
    priority_queue<LiseNode*, vector<ListNode*>, Compare>min_heap;
    
    for(ListNode* list : lists){
        if(list != nullptr)
            min_heap.push(list);
    }
    
    ListNode *dummy = new ListNode(); // 哨兵
    LiseNode *cur = dummy; 
    while(!min_heap.empty()){
        ListNode *min_node = min_heap.top();
        min_heap.pop();
        
        cur->next = min_node;
        cur = cur->next;
        
        if(min_node->next)
            min_heap.push(min_node->next);
    }
    return dummy->next;
}
```

#### 24 两两交换链表中的节点

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
void swap(int &a, int &b){
    int tem = a;
    a = b;
    b = tem;
}

ListNode* swapPairs(ListNode* head){
    if(!head || !head->next)
        return head;
    ListNode* p = head;
    ListNode* r = p->next;
    int count = 0;
    while(r){
        ++count;
        if(count & 1){
            swap(p->val, r->val);
        }
        p = p->next;
        r = r->next;
    }
    return head;
}
```

#### 25 K个一组翻转链表

```c++
/*
在这个解决方案中，我们首先计算链表的长度。然后，我们迭代地将每个长度为k的子链表翻转。为了简化边界条件，我们使用一个dummy节点。在每次循环中，我们将一个节点移动到子链表的头部，然后更新相应的指针。这个方法的时间复杂度是O(n)，其中n是链表的节点数。
*/
ListNode* reverseKGroup(ListNode* head, int k){
    if(!head || k == 1)
        return head;
    ListNode *dummy = new ListNode(0, head);
    ListNode *cur = head;
    ListNode *prev = dummy;
    int len = 0;
    while(cur){
        ++len;
        cur = cur->next;
    }
    while(len >= K){
        ListNode *subListTail = prev;
        for(int i = 0; i < k; ++i){
            //头插法
            ListNode* nextNode = cur->next;
            cur->next = subListTail->next;
            subListTail->next = cur;
            cur = nextNode;
        }
        for(int i = 0; i<k; ++i)
            prev = prev->next;
        prev->next = cur;
        len -= k;
    }
    return dummy->next;
}

```

#### 26 删除数组的重复项

```c++
int removeDuplicates(vector<int>& nums){
	int n = nums.size();
    int i=0,j= i+1;
    while(j < n){
        if(nums[i] != nums[j])
            nums[++i] = nums[j];
        ++j;
    }
    return i+1;
}
```

#### 27 移除元素

```c++
int removeElement(vector<int>& nums, int val){
    int n = nums.size();
    int i = 0, k= 0;
    while(i < n){
        if(nums[i] != val)
            nums[k++] = nums[i];
        ++i;
    }
    return k;
}
```

#### 28 找出字符串中第一个匹配项的下标

```c++
// 暴力
 int strStr(string haystack, string needle){
     int i = 0, m = haystack.size();
     int j = 0, n = needle.size();
     
     for(int i = 0; i < n; ++i){
         int k = i, j = 0;
         while(haystack[k] == needle[j] && j < n){
             ++K;
             ++j;
         }
         if(j == n){
             return i;
         }
     }
     return -1;
 }
//kmp
vector<int> computePrefixFunction(const string& pattern){
    int m = pattern.size();
    vector<int>pi(m);
    pi[0] = 0;
    int k = 0;
    
    for(int q = 1; q < m; ++q){
        while(k > 0 && pattern[k] != pattern[q]){
            k = pi[k -1];
        }
        
        if(pattern[k] == pattern[q]){
            ++k;
        }
        pi[q] = k;
    }
    return pi;
}

int strStr(string haystack, string needle){
    int n = haystack.size();
    int m = needle.size();
    
    if(m == 0)
        return 0;
    
    vector<int> pi = computePrefixFunction(needle);
    int q = 0;
    
    for(int i = 0; i < n; ++i){
        while(q > 0 && needle[q] != haystack[i]){
            q = pi[q -1];
        }
        if(needle[q] == haystack[i])
            ++q;
        if(q == m)
            return i - m + 1;
    }
    return -1;
}
```

#### 29 两数相除(左移)

```c++
/*
这个实现使用了位操作（左移）来实现除法。首先将被除数和除数转换为绝对值并存储为长整型，然后通过迭代过程，每次将除数乘以2的幂次（通过左移实现）直到它大于被除数。接下来，将被除数减去当前的除数值，并将结果加到结果中。最后根据符号返回结果。
*/
int divide(int dividend, int divisor){
    if(dividend == INT_MIN && divisor == -1){
        return INT_MAX;
    }
    long long dvd = labs(static_cast<long long>(dividend)); // labs 是一种求long类型的绝对值函数
    long long dvs = labs(static_cast<long long>(divisor));
    int sign = (dividend > 0) ^ (divisor > 0) ? -1:1; // ^ 按位异或  
    //用 dividend > 0 为true 还是 false  和 divisor > 0 为true 还是 false 做异或
    // 同为true 同为false 为 false， 一正一负 为true
    long long res = 0;
    
    while(dvd >= dvs){
        long long temp = dvs;
        long long multiple = 1;
        while(dvd >= (temp << 1)){
            temp <<= 1;
            multiple <<= 1;
        }
        dvd -= temp;
        res += multiple;
    }
    return sign == 1 ? res : -res; 
}
```

#### 30 串联所有单词的子串

```c++
vector<int> findSubstring(string s, vector<string>& words){
    if(words.empty())
        return{};
    unordered_map<string,int> word_count;
    for(const auto& word:words){
        ++word_count[word];
    }
    
    int word_len = words[0].length();
    int word_total_len = word_len * words.size();
    int s_len = s.length();
    vector<int> result;
    
    for(int i = 0; i < s_len - word_total_len; ++i){
        unordered_map<string, int>substring_count;
        int j = 0;
        
  		while(j < words.size()){
            string current_word = s.substr(i + j * word_len, word_len);
            if(word_count.find(current_word) == word_count.end()){
                break;
            }
            ++substring_count[current_word];
            if(substring_count[current_word] > word_count[current_word]){
                break;
            }
            ++j;
        }
        
        if(j == words.size()){
            result.push_back(i);
        }
    }
    return reslut;
}
```

#### 31 下一个排列

```c++
/*
首先从右向左找到第一个升序对，然后在该位置上找到一个稍大的数字进行交换。最后，反转子序列以获得下一个排列。
*/
void nextPermutation(vector<int>& nums){
    int n = nums.size();
    int i = n - 2;
    while(i >= 0 && nums[i] >= nums[i + 1]) --i;
    if(i >= 0){
        int j = n -1;
        while(j > i && nums[j] <= nums[i]) --j;
        swap(nums[i], nums[j]);
    }
    
    reverse(nums.begin() + i + 1, nums.end());
}
```

#### 32 最长有效括号(stack)

```c++
/*
这个解决方案使用了一个栈来保存括号的索引。遍历输入字符串，当遇到左括号时，将其索引入栈；当遇到右括号时，出栈。在这个过程中，通过计算当前索引与栈顶索引之间的距离来更新最长有效括号子串的长度。
*/
int longestValidParentheses(string s){
    int maxLength = 0;
    stack<int> st;
    st.push(-1);
    
    for(int i = 0; i < s.size(); ++i){
        if(s[i] == '('){
            st.push(i);
        }else{
            st.pop();
            if(st.empty()){
                st.push(i);
            }else{
                maxLength = max(maxLength, i - st.top());
            }
        }
    }
    return maxLength;
}
```



#### 33.搜索旋转排序数组 (二分)

```c++
int search(vector<int>& nums, int target){
    int l = 0, r = nums.size() -1;
    while(l <= r){
        int mid = (l + r) >> 1;
        if(nums[mid] == target)
            return mid;
       	if(nums[mid] >= nums[l]){ // 左半有序
            if(target >= nums[l] && target < nums[mid])
                j = mid - 1;
            else
                i = mid + 1;
        }else{ //右半有序
            if(target > nums[mid] && target <= nums[j])
                i = mid + 1;
            else
                j = mid - 1;
        }
    }
    return -1;
}
```



#### 34在排序数组中查找元素的第一个和最后一个位置（二分）

```c++
vector<int> searchRange(vector<int>& nums, int target) {
    return{searchRangeBounder(nums, target, "left"), searchRangeBounder(nums, target, "right")};
}

int searchRangeBounder(vector<int>& nums, int target, const string& bound){
    int l = 0, r = nums.size() - 1;
    int mid, res = -1;
    while(l <= r){
        mid = (l + r) >> 1; // 相当于(l + r) /2
        if(nums[mid] < target)
            l = mid + 1;
        else if(nums[mid] > target)
            r = mid - 1;
        else{
            res = mid;
            if(bound == "left")
                r = mid - 1;
            else if(bound == "right")
                l = mid + 1;
            else{

            }
        }
    }
    return res;
}
```

#### 35 搜索插入位置（二分）

```c++
int searchInsert(vector<int>& nums, int target){
    int left = 0, right = nums.size() - 1;
    
    while(left <= right){
        int mid = left + (right - left) /2; // 最好用这种方式，用右移容易导致下标溢出
        if(nums[mid] == target){
            return mid;
        }else if(nums[mid] > target){
            right = mid - 1;
        }else{
            left = mid + 1;
        }
    }
    return left;
}
```



#### 36 有效数独

```c++
/*
这个解决方案使用了三个二维布尔向量来分别跟踪每行、每列和每个 3x3 子数独中已存在的数字。遍历整个数独板，检查当前位置的数字是否已经存在于对应的行、列或子数独中。如果存在，则返回 false。如果遍历完成且没有发现重复的数字，则返回 true。
*/    
bool isValidSudoku(vector<vector<char>>& board) {
        vector<vector<bool>> rows(9, vector<bool>(9, false));
        vector<vector<bool>> cols(9, vector<bool>(9, false));
        vector<vector<bool>> boxes(9, vector<bool>(9, false));

        for(int i = 0; i < 9; ++i){
            for(int j = 0; j < 9; ++j){
                if(board[i][j] !='.'){
                    int num = board[i][j] - '1'; // 将字符串转换成整数
                    //计算 3 * 3 子数独的索引
                    int boxIndex = (i / 3) * 3 + j / 3;
                    //检查行、列和子数独中是否已经存在该数字
                    if(rows[i][num] || cols[j][num] || boxes[boxIndex][num])
                    return false; 

                    rows[i][num] = true;
                    cols[j][num] = true;
                    boxes[boxIndex][num] = true;

                }
            }
        }
        return true;

    }
```

#### 37 解数独（回溯）

```c++
/*这个解法使用了回溯法。对于每一个空格，尝试填入 '1' 到 '9'，然后对每一个尝试继续尝试解决数独，如果不能解决，就撤销这一步，并继续尝试下一个数字。我们可以通过这种方式找到所有可能的数独解。
*/
void solveSudoku(vector<vector<char>>& board) {
	solve(board);
}

bool solve(vector<vector<char>>&board){
	for(int i = 0; i < 9; ++i){
        for(int j = 0; j < 9; ++j){
            if(board[i][j] == '.'){
                for(char c ='1'; c <= '9'){
                    if(isVaild(board, i ,j, c){
                        board[i][j] = c;
                        
                        if(solve(board)){
                            return true;
                        }
                        
                        board[i][j] = '.';
                    }
                }
                return false;
            }
        }
    }
     return true;
}
                       
bool isValid(vector<vector<char>>& board, int row, int col, char c){
    for(int i = 0; i < 9; ++i){
        if(board[i][col] == c){
            return false;
        }
        if(boar[row][i] == c)
            return false;
        if(board[3 * (row/3) + i / 3][3 *(col/ 3)+ i % 3] == c)
            return false;
    }
    return true;
}
```

#### 38 外观数列(递归)

```c++
/*
这个解法使用了递归。对于 n > 1，首先计算 n-1 的结果，然后对于结果中的每个字符，计算它的连续出现次数，然后将计数和字符添加到结果中。
*/
string countAndSay(int n){
	if(n == 1){
		return "1";
	}
    
    string prev = countAndSay(n -1);
    string res = "";
    
    int count = 1;
    for(int i = 0; i < prev.size(); ++i){
        //如果当前的数字和下一个数字相同，增加计数
        if(i < prev.size() - 1 && prev[i] == prev[i + 1]){
            ++count;
        }else{
            //否则。将当前计数和数字添加到结果中
            res += to_string(count) + prev[i];
            count = 1; //重置计数
        }
    }
    return res;
}
```



#### 39 组合总和 （dfs）

```c++
/*
该代码使用深度优先搜索（DFS）解决问题。主要思路是，对于每个数，我们可以选择将其加入当前组合，或者不加入。如果加入了这个数，我们仍然可以从当前的数集中选择，如果没有选择该数，我们可以从它的下一个位置开始选择。我们可以使用一个循环遍历所有可能的选择，并使用递归来继续搜索。

具体来说，我们维护一个当前组合curr和一个start变量，表示我们从候选数组中的哪个位置开始搜索。在搜索时，我们检查当前的目标是否为0。如果是，则我们找到了一个组合，将其添加到结果中并返回。否则，如果当前目标小于0，我们应该停止搜索并返回。对于每个候选数，我们将其添加到当前组合中，并递归搜索剩余目标。当搜索返回时，我们将候选数从当前组合中删除，并尝试其他可能的选择。

该代码的时间复杂度为$O(N^t)$，其中N是候选数组的长度，t是目标数值的大小。由于我们必须要访问所有的可能组合，所以这个时间复杂度是无法避免的。空间复杂度为$O(t)$，因为我们需要保存每个组合。
*/
vector<vector<int>>combinationSum(vector<int>& candidates, int target){
    sort(candidates.begain(), candidates.end());
    vector<vector<int>>res;
    vector<int> curr;
    function<void(int, int)>dfs = [&](int i, int t){ // 返回类型是 void， 匿名函数
      if(t == 0)
      {
          res.emplace_back(curr);
          return;
      }
      if(t < candidates[i])
          return;
      for(int j = i; j < candidates.size(); ++j)
      {
          curr.emplace_back(candidates[j]);
          dfs(j, t - candidates[j]);
          curr.pop_back();
      }
    };
    dfs(0, target);
    return res;
}
```

#### 40 组合总和II（回溯）

```c++
/*
	这个解法使用了回溯算法和剪枝。在寻找所有可能的组合的过程中，我们首先对候选数组进行排序，然后通过回溯寻找所有满足条件的组合。在回溯过程中，我们需要跳过重复的元素以避免出现重复的组合。另外，如果当前候选元素大于目标值，我们就可以停止回溯，因为在一个有序数组中，如果当前元素已经大于目标值，那么后面的元素肯定也大于目标值。
*/
vector<vector<int>> combinationSum2(vector<int>& candidates, int target){
    sort(candidates.begin(), candidates.end());
    vector<vector<int>> res;
    vector<int>combination;
    backtrack(candidates, target, res, combination, 0);
    return res;
}

void backtrack(vector<int>& candidates, int target, vector<vector<int>>& res, vector<int>& combination, int begin){
    if(target == 0){
        res.emplace_back(combination);
        return;
    }
    
    for(int i = 0; i < candidates.size() && target > candidates[i]； ++i){
        if(i == begin || candidates[i] != candidates[i - 1]){
            combination.push_back(candidates[i]);
            backtrack(candidates, target-candidates[i], res, combination, i+1);
            combination.pop_back();
        }
    }
}
```

#### 41 缺失的第一个正数

```c++
/*
这个解决方案的主要思想是尽可能地将每个正数放在数组中它应该在的位置上，也就是 nums[i] 应该等于 i+1。然后，再次遍历数组，找到第一个不在应在位置的元素。这个元素就是第一个缺失的正数。如果数组中的所有元素都在它们应该在的位置，那么第一个缺失的正数就是 n+1。
*/
int firstMissingPositive(vector<int>& nums){
    int n = nums.size();
    
    //遍历数组，将值为i的元素调整到第i个的位置上
    for(int i = 0; i < n; ++i){
        while(nums[i] > 0 && nums[i] < n && nums[nums[i] - 1] != nums[i]){
            swap(nums[i], nums[nums[i] - 1]);
        }
    }
    
    //在次遍历数组， 找到一个不应在位置上的元素
    for(int i = 0; i < n; ++i){
        if(nums[i] != i + 1)
            return i + 1;
    }
    
    //如果都在应在的位置。返回n + 1
    return n + 1;
}
```

#### 42 接雨水（stack）

```c++
/*
此解法使用了单调栈。首先，我们从左到右遍历数组，对于每一个元素，我们尝试清空栈直到当前元素小于栈顶元素。在这个过程中，我们根据木桶效应计算出每一层可以储存的水，并累加到最终结果中。在清空栈的过程中，我们保证栈内的元素在数组中是单调非递增的。当遍历完整个数组后，我们就可以得到储水的总量。
*/
int trap(vector<int>& height) {
    int res = 0, cur = 0;
    satck<int> st;
    int n = height.size();
    while(cur < n){
        while(!st.empty() && height[cur] >= height[st.top()]){
            int top = st.top();
            st.pop();
            if(st.empty())
                break;
            int distance = current - st.top() -1;
            int bounded_heigt = min(height[cur], height[st.top()] - height[top]);
            res += bounded_height * distance;
        }
    }
}

```

#### 43 字符串相乘

```c++
/*
这个解法的思路是对于 num1 和 num2 的每一位，我们都计算其乘积并添加到相应的位置上。对于 num1 的第 i 位和 num2 的第 j 位，其乘积将会被添加到结果的第 i+j 和 i+j+1 位上。这样我们就可以模拟人类手动计算乘法的步骤，得到乘法的结果。
*/
string multiply(string num1, string num2){
	if(num1 == "0" || num2 == "0"){
        return "0";
    }	
    
    int n = num1.size(), m=num2.size();
    vector<int>res(n+m, 0);
    
    for(int i = n-1; i >= 0; --i){
        int n1 = num1[i] - '0';
        for(int j = m-1; j >= 0; --j){
            int n2 = num2[j] -'0';
            int sum = (res[i+j+1] + n1 * n2);
            res[i+j+1] = sum %10;
            res[i+j] += sum /10;
        }
    }
    
    int i = 0;
    while(i < res.size() && res[i] == 0){
        ++i;
    }
    
    string str;
    for(;i<res.size(); ++i){
        str.push_back('0'+res[i]);
    }
    return str;
}
```

#### 44 通配符匹配（动态规划）

```c++
/*
这个解法使用了动态规划。dp[i][j] 表示 s 的前 i 个字符和 p 的前 j 个字符是否匹配。对于 p 的第 j 个字符，如果它是 '*'，那么就有两种情况：匹配零个字符，对应 dp[i][j] = dp[i][j - 1]；匹配一个或多个字符，对应 dp[i][j] = dp[i - 1][j]。如果它不是 '*'，那么就看 s 的第 i 个字符和 p 的第 j 个字符是否匹配，如果匹配，那么 dp[i][j] = dp[i - 1][j - 1]。最后的结果就是 dp[m][n]，其中 m 和 n 是 s 和 p 的长度
*/
bool isMatch(string s, string p){
    int m = s.size(), n=p.size();
    vector<vector<bool>>dp(m+1, vector<bool>(n+1, false));
    dp[0][0] = true;
    
    for(int i = 1; i <= n; ++i){
        if(p[i -1] == '*'){
            dp[0][i] = true;
        }else{
            break;
        }
    }
    
    for(int i = 1; i <= m; ++i){
        for(int j = 1; j <= n; ++j){
            if(p[j -1] == '*'){
                dp[i][j] = dp[i][j-1] || dp[i-1][j];
            }else if(p[j -1] == '?' || s[i -1] == p[j -1]){
                dp[i][j] = dp[i-1][j-1];
            }
        }
    }
    return dp[m][n];
}

```

#### 45 跳跃游戏II（贪心）

```c++
/*
这个解法的主要思想是贪心算法。我们从左到右遍历数组，对于每一个位置，我们都计算能够跳到的最远的位置。如果当前位置达到了之前计算的最远的位置，那么就进行一次跳跃，并更新最远的位置。这样，当我们遍历完整个数组后，就可以得到最少的跳跃次数。
*/
int jump(vector<int>& nums){
    int n = nums.size();
    int end = 0, farthest = 0;
    int jumps = 0;
    for(int i = 0; i < n -1; ++i){
        farthest = max(nums[i] + i, farthest);
        if(end == i){
            ++jumps;
            end = farthest;
        }
    }
    return jumps;
}
```

#### 46 全排列（回溯）

```c++
/*
这个解法使用了回溯算法来生成所有的排列。对于每一个位置，我们都尝试将它和后面的每一个位置进行交换，然后递归地生成后面位置的所有排列，最后再将这两个位置交换回来。这样，我们就能够生成所有的排列。
*/

vector<vector<int>> permute(vector<int>& nums){
    vector<vector<int>>res;
    backtrack(nums, 0, res);
    return res;
}

void backtrack(vector<int>& nums, int start, vector<vector<int>>& res){
    if(start == nums.size()){
        res.emplace_back(nums);
        return;
    }
    for(int i = start; i < nums.size(); ++i){
        swap(nums[start], nums[i]);
        backtrack(nums, start + 1, res);
        swap(nums[start], nums[i]);
    }
}
```

#### 47 全排列II(回溯)

```c++
vector<vector<int>> permuteUnique(vector<int>& nums){
    sort(nums.begin(), nums.end());
    vector<vector<int>>res;
    backtrack(nums, 0, res);
    return res;
}

void backtrack(vector<int>&nums, int start, vector<vector<int>>& res){
    if(start == nums.size()){
        res.emplace_back(nums);
        return;
    }
    for(int i = start; i < nums.size(); ++i){
        if(i != start && checkDuplicate(nums, start, i)) continue;
        swap(nums[start], nums[i]);
        backtrack(nums, satrt + 1, res);
        swap(nums[start], nums[i]);
    }
}
```

#### 48 旋转图像

```c++

void rotate(vector<vector<int>>& matrix){
    int n = matrix.size();
    for(int i = 0; i < n; ++i){
        for(int j = i; j < n; ++j){
            swap(matrix[i][j],matrix[j][i]);
        }
    }
    
    for(int i = 0; i < n; ++i){
        for(int j = 0; j < n /2; ++j){
            swap(matrix[i][j], matrix[i][n-j-1]);
        }
    }
}
```

#### i49 字母异位分词 (哈希表)

```c++
/*
这个解法的主要思想是将所有的字符串按照字母的顺序排序，然后将排序后的字符串作为哈希表的键，原字符串作为哈希表的值。这样，所有字母相同的字符串就会被放在同一个键下面，从而实现对字母相同的字符串的分组。
*/
vector<vector<string>> groupAnagrams(vector<string>& strs){
	unordered_mp<string, vector<string>>mp;
    for(string&s:strs){
        string t = s;
        sort(t.begin(), t.end());
        mp[t].emplace_back(s);
    }
    
    vector<vector<string>> anagrams;
    for(auto& n : mp){
        anagrams.emplace_back(n.second);
    }
    return anagrams;
}
```

#### 50 Pow(x ,n）

```c++
/*
这个解法使用了快速幂算法。在每一步中，我们都将幂除以 2，然后将底数平方，如果幂是奇数，那么我们就将结果乘以底数。这样，我们就可以在 log(n) 的时间复杂度内计算出结果。
*/

double myPow(double x , int n){
    long long N = n;
    if(N < 0){
        x = 1 / x;
        N = -N;
    }
    double ans;
    double current_product = x;
    for(long long i = N; i; i /= 2){
        if((i % 2) == 1)
        {
            ans *= current_product;
        }
        current_product *= current_product;
    }
    return ans;
}
```

#### 51 N皇后（dfs）

```c++
/*
这里采用了深度优先搜索（DFS）的思想，在递归过程中搜索合法的解。对于每一行，逐列搜索合法的位置，如果找到了合法的位置，则将皇后放置在该位置，然后继续搜索下一行。如果在某一行没有找到合法的位置，则回溯到上一行，并撤销之前放置的皇后，继续搜索下一列。当搜索到最后一行时，说明找到了一个合法的解，将其保存下来。
*/

vector<vector<string>> solveNQueens(int n){
    vector<vector<string>> & result;
    vector<string> solution(n, string(n, '.')); //初始化 n*n 棋盘
    
    solveNQueensDFS(solution, result, 0, n);
    return result;
}

void solveNQueensDFS(vector<string>& solution, vector<vector<string>>& result, int row, int n){
    if(row == n){	// 找到一个解
        result.emplace_back(solution);
        return;
    }
    for(int col = 0; col < n; ++col){	// 逐列搜索合法位置
        if(isValid(solution, row, col, n)){ // 判断是否是合法位置
            solution[i][j] = 'Q';	// 放置皇后
            solveNQueensDFS(solution, result, row+1, n); // 继续搜索下一行
            solution[i][j] = '.';	// 回溯，撤销放置
        }
    }
}

bool isValid(vector<string>& solution, int row, int col, int n){
    for(int i = 0; i < row; ++i){	// 判断同一列是否有皇后
        if(solution[i][col] == 'Q'){
            return false;
        }
    }
    
    for(int i = row-1, j = col-1; i >= 0 && j >= 0; --i,--j){	//  判断左上方是否有皇后
        if(solution[i][j] == 'Q'){
            return false;
        }
    }
    
    for(int i = row-1, j = col + 1; i >= 0 && j < n; --i, ++j){	// 判断右上方是否有皇后
        if(solution[i][j] == 'Q'){
            return false;
        }
    }
}
```

#### 52 N皇后II （dfs）

```c++
/*
这里采用了深度优先搜索（DFS）的思想，在递归过程中搜索合法的解。对于每一行，逐列搜索合法的位置，如果找到了合法的位置，则标记该列、该行的左上到右下的对角线和该行的右上到左下的对角线上有皇后，并继续搜索下一行。如果在某一行没有找到合法的位置，则回溯到上一行，并撤销之前标记的皇后位置，继续搜索下一列。当搜索到最后一行时，说明找到了一个合法的解，将计数器加1。
*/

/*
52 和 51 都是 N 皇后问题，但是求解的方式略有不同。51 需要求出所有可能的解法，而 52 只需要求出解法的总数。
*/
class Solution {
public:
    int totalNQueens(int n) {
        int count = 0;
        vector<bool> cols(n, false); // 记录每列是否有皇后
        vector<bool> diag1(2*n-1, false); // 记录左上到右下的对角线是否有皇后
        vector<bool> diag2(2*n-1, false); // 记录右上到左下的对角线是否有皇后

        totalNQueensDFS(count, cols, diag1, diag2, 0, n);

        return count;
    }

    void totalNQueensDFS(int& count, vector<bool>& cols, vector<bool>& diag1, vector<bool>& diag2, int row, int n) {
        if (row == n) { // 找到一个解
            ++count;
            return;
        }

        for (int col = 0; col < n; ++col) { // 逐列搜索合法位置
            if (!cols[col] && !diag1[row+col] && !diag2[n-1-row+col]) { // 判断是否是合法位置
                cols[col] = true; // 标记该列有皇后
                diag1[row+col] = true; // 标记左上到右下的对角线有皇后
                diag2[n-1-row+col] = true; // 标记右上到左下的对角线有皇后
                totalNQueensDFS(count, cols, diag1, diag2, row+1, n); // 继续搜索下一行
                cols[col] = false; // 回溯，撤销标记
                diag1[row+col] = false; // 回溯，撤销标记
                diag2[n-1-row+col] = false; // 回溯，撤销标记
            }
        }
    }
};

```

#### 53 最大子数组和 (动态规划)

```c++
/*
可以采用动态规划的思想来解决这个问题。定义一个 dp 数组，其中 dp[i] 表示以第 i 个元素结尾的连续子数组的最大和。显然，dp[0] = nums[0]。然后，对于 i > 0，如果 dp[i-1] > 0，则 dp[i] = dp[i-1] + nums[i]，否则 dp[i] = nums[i]。
*/

int maxArray(vector<int> nums){
    int n = nums.size();
    if(n == 0){
        return 0;
    }
    int dp = nums[0];
    int maxSum = 0;
    
    for(int i = 1; i < n; ++i){
        dp = max(dp+nums[i], nums[i]);
        maxSum = max(maxSum, dp);
    }
    return maxSum;
}
```

#### 54 螺旋矩阵

```c++
/*
具体来说，我们可以定义四个变量，left、right、top、bottom，分别表示当前需要输出的矩阵的左、右、上、下边界。然后循环输出矩阵中的每一个元素，直到 left > right 或 top > bottom。
*/
vector<int> spiralOrder(vector<vector<int>>& matrix)}{
	vector<int>res;
    int m = matrix.size(); //行数
    if(m == 0){
        return res;
    }
    
    int n = matrix[0].size();  //列数
    int left = 0, right = n -1, top = 0, bottom = m - 1;
    
    while(left <= right && top <= bottom){
        
        //从左到右
        for(int i = left; i <= right; ++i){
            res.push_back(matrix[top][i]);
        }
        
        //从上到下
        for(int i = top+1; i <= bottom; ++i){
            res.push_back(matrix[i][right]);
        }
        
        //从右至左
        if(top < bottom){
            for(int i = right-1; i >=left; --i){
                res.push_back(matrix[bottom][i]);
            }
        }
        
        //从下至上
        if(left < right){
        	for(int i= bottom -1; i > top; --i){
                res.push_back(matrix[i][left]);
            }
        }
        ++left, ++top;
        --right, --bottom;
    }
    return res;
}
```

#### 55 跳跃游戏 

```c++
/*
在这个代码中，我们使用了一个变量 max_pos 来记录当前能够到达的最远位置，初始化为0。然后遍历数组中的每一个元素，如果当前位置 i 小于等于 max_pos，那么就更新 max_pos 的值，max_pos 更新为 max(max_pos, i + nums[i])，表示当前位置能够到达的最远位置。如果在更新 max_pos 的过程中，max_pos 的值已经大于等于 n - 1，那么就说明可以到达最后一个位置，直接返回 true。如果遍历完整个数组之后，max_pos 的值仍然小于 n - 1，那么就说明无法到达最后一个位置，返回 false。
*/
bool canJump(vector<int>& nums){
	int max_pos = 0;
    int n = nums.size();
    for(int i=0; i < n; ++i){
        if(i <= max_pos){
            max_pos = max(max_pos, i+nums[i]);
            if(max_pos >= n-1){
                return ture;
            }
        }
    }
    return false;
}
```

#### 56 合并区间

```c++
/*
首先，对区间按照起始位置进行排序，然后使用一个新的结果集合 merged 来存储合并后的区间。

我们遍历排序后的区间列表，对于每个区间 interval，如果 merged 为空或者当前区间与 merged 中的最后一个区间没有重叠，那么将当前区间加入 merged。

如果当前区间与 merged 中的最后一个区间重叠，那么更新 merged 中的最后一个区间的结束位置为当前区间的结束位置和 merged 中最后一个区间的结束位置的较大值。
*/
vector<vector<int>> merge(vector<vector<int>>& intervals){
    sort(intervals.begin(), intervals.end(), [](vector<int>& a, vector<int>& b){return a[0] < b[0];});
    vector<vector<int>>merged；
    for(auto & interval : intervals){
        if(merged.empty() || interval[0] > merged.back()[1]){
            merged.emplace_back(interval);
        }else{
            merged.back()[1] = max(merged.back()[1], interval[1]);
        }
    }
    return merged;
}
```

#### 57 插入区间

```c++
/*

*/
vector<vector<int>> insert(vector<vector<int>>& intervals, vector<int>&newInterval){
    vector<vector<int>>& merged;
    int i = 0;
    int n = intervals.size();
    while(i < n && intervals[i][1] < intervals[0]){
        merged.emplace_back(intervals[i++]);
    }
    
    while(i < n && interval[i][0] <= newInterval[1]){
        newInterval[0] = min(newInterval[0], intervals[i][0]);
        newInterval[1] = max(newInterval[1], intervals[i][1]);
        ++i;
    }
    merged.emplace_back(newInterval);
    while(i < n){
        merged.emplace_back(intervals[i++]);
    }
    return merged;
}
```

#### 58 最后一个单词的长度

```c++
/*
然后，我们从字符串 s 的末尾开始往前遍历，找到第一个非空格字符的位置。

接下来，我们继续往前遍历，直到遇到空格字符或者到达字符串的开头。
*/
int lengthlastWord(string s){
    int len = 0;
    int i = s.length() -1;
    
    while(i >= 0 && s[i] == ' '){
        --i;
    }
    while(i >= 0 && s[i] != ' '){
        ++len;
        --i;
    }
    return len;
}
```

#### 59 螺旋矩阵II

```c++
vector<vector<int>>generateMatrix(int n){
	vector<vector<int>>matrix(n vector<int>(n, 0));
    int num = 1;
    int top = 0, bottom = n-1;
    int left = 0, right = n-1;
    while(top <= bottom && left <= right){
        for(int i = left; i <= right; ++i){
            matrix[top][i] = num++;
        }
        ++top;
        
        for(int i = top; i <= bottom; ++i){
            matrix[i][right] = num++;
        }
        --right;
        
        if(top <= bottom){
            for(int i = right; i >= left; --i){
                matrix[bottom][i] = num++;
            }
            --bottom;
        }
        
        if(left <= right){
            for(int i = bottom; i >= top; --i){
                matrix[i][left] = num++;
            }
            ++left;
        }
    }
    return matrix;
}
```

#### 60 排列序列

```c++
/*
利用阶乘数来计算第 k 个排列。我们先创建一个存储阶乘数的数组 factorial，然后将 k 转换为从0开始的索引，即 k--。

接下来，从最高位开始，依次计算每个位置上的数字。对于每个位置，我们通过 k / factorial[i] 的结果来确定当前位置的数字在可用数字集合中的索引。然后，我们将该数字加入结果字符串 result 中，从 nums 中移除，并更新 k 的值为 k % factorial[i]。
*/

string getPermutation(int n, int k){
    string result;
    vector<int>nums;
    
    for(int i = 1; i <= n; ++i){
        nums.push_back(i);
    }
    
    vector<int> factorial(n ,1);
    for(int i = 1; i < n; ++i){
        factorial[i] = factorial[i - 1] * i;
    }
    
    //转换为从0开始的索引
    --k;
    
    for(int i = n -1; i >= 0; ++i){
        int index = k / factorial[i];
        result += to_string(nums[index]);
        nums.erase(nums.begin()+index);
        k %= factorial[i];
    }
    return result;
}
```

#### 61  旋转链表

```c++
ListNode* rotateRight(ListNode* head, int k) {
    if (!head || !head->next || k == 0)
        return head;
    
    int len = 1;  // 链表长度
    ListNode* tail = head;
    while (tail->next) {
        len++;
        tail = tail->next;
    }
    
    k %= len;  // 处理 k 大于链表长度的情况
    if (k == 0)
        return head;
    
    // 找到新的头节点的前一个节点
    ListNode* newHeadPrev = head;
    for (int i = 1; i < len - k; i++) {
        newHeadPrev = newHeadPrev->next;
    }
    
    ListNode* newHead = newHeadPrev->next;
    newHeadPrev->next = nullptr;  // 断开链表
    
    tail->next = head;  // 将链表的尾部节点连接到原头节点
    
    return newHead;
}

```

#### 62 不同路径（动态规划）

```c++
/*
我们使用一个二维数组 dp 来保存不同位置的路径数。在两个嵌套的循环中，我们根据递推关系更新 dp 数组中的值。最后返回 dp[m-1][n-1] 即可得到总共的不同路径数。
*/ 
int uniquePaths(int m, int n){
     vector<vector<int>>dp(m, vector<int>(n, 1));
     
     for(int i = 1; i < m; ++i){
         for(int j = 1; j < n; ++j){
             dp[i][j] = dp[i-1][j] + dp[i][j-1]
         }
     }
 }
```

#### 63 不同路径II（动态规划）

```c++
/*
动态规划的递推关系如下：

对于第一行和第一列，由于机器人只能向右或向下移动，如果当前位置没有障碍物，则到达这些位置的路径数等于它左方位置的路径数或上方位置的路径数（如果有障碍物，则路径数为 0），即 dp[i][0] = obstacleGrid[i][0] == 1 ? 0 : dp[i-1][0] 和 dp[0][j] = obstacleGrid[0][j] == 1 ? 0 : dp[0][j-1]。


对于其他位置 (i, j)，如果当前位置没有障碍物，则到达该位置的路径数等于它上方位置 (i-1, j) 的路径数与左方位置 (i, j-1) 的路径数之和（如果有障碍物，则路径数为 0），即 dp[i][j] = obstacleGrid[i][j] == 1 ? 0 : dp[i-1][j] + dp[i][j-1]。

*/
int uniquePathWithObstacles(vector<vector<int>>& obstacleGrid){
    int m = obstacleGrid.size();  //行数
    int m = obstacleGrid[0].size(); // 列数
    
    //初始化第一行第一列
    
    for(int i = 0; i < m; ++i){
        if(obstacleGrid[i][0] == 0){
            dp[i][0] == 1;
        }else{
            break;
        }
    }
    
    for(int j = 0; j < n; ++j){
        if(obstacleGrid[0][j] == 0){
            dp[0][j] =1;
        }else{
            break;
        }
    }
    
    for(int i = 1; i < m; ++i){
        for(int j = 1; j < n; ++j){
            if(obsatcleGrid[i][j] == 0){
                dp[i][j] = dp[i-1][j] + dp[i][j-1];
            }
        }
    }
    return dp[m-1][n-1];
}
```



#### 64 最小路径和 (动态规划)

```c++
/*
动态规划的递推关系如下：

对于第一行和第一列，由于机器人只能向右或向下移动，所以到达这些位置的最小路径和等于前一个位置的最小路径和加上当前位置的值，即 dp[i][0] = dp[i-1][0] + grid[i][0] 和 dp[0][j] = dp[0][j-1] + grid[0][j]。


对于其他位置 (i, j)，到达该位置的最小路径和等于上方位置 (i-1, j) 和左方位置 (i, j-1) 中较小的路径和加上当前位置的值，即 dp[i][j] = min(dp[i-1][j], dp[i][j-1]) + grid[i][j]。
*/

int minPathSum(vector<vector<int>> & grid){
    int m = grid.size(); //行数
    int n = grid.size(); //列数
    vector<vector<int>>dp(m, vector<int>(n, 0));
    
    dp[0][0] = grid[0][0];
    
    //初始化第一行第一列
    for(int i = 1; i < m; ++i){
        dp[i][0] = dp[i-1][0] + grid[i][0];  //初始化第一列
    }
    
    for(int j = 1; j < n; ++j){
        dp[0][j] = dp[0][j-1] + grid[0][j];
    }
    
    for(int i = 1; i < m; ++i){
        for(int j = 1; j < n; ++j){
            dp[i][j] = min(dp[i-1][j],dp[i][j-1]) + grid[i][j];
        }
    }
    return dp[m-1][n-1]
}
```

#### 65 有效数字（有限状态机）

```c++
/*
这段代码使用状态机的方法来判断一个字符串是否为有效数字。代码中定义了两个枚举类型，State 表示状态，CharType 表示字符类型。通过状态转移图进行状态的切换，最终判断最后的状态是否为有效的结束状态。
*/
class Solution {
public:
    enum State {
        STATE_INITIAL,
        STATE_INT_SIGN,
        STATE_INTEGER,
        STATE_POINT,
        STATE_POINT_WITHOUT_INT,
        STATE_FRACTION,
        STATE_EXP,
        STATE_EXP_SIGN,
        STATE_EXP_NUMBER,
        STATE_END
    };

    enum CharType {
        CHAR_NUMBER,
        CHAR_EXP,
        CHAR_POINT,
        CHAR_SIGN,
        CHAR_ILLEGAL
    };

    CharType toCharType(char ch) {
        if (ch >= '0' && ch <= '9') {
            return CHAR_NUMBER;
        } else if (ch == 'e' || ch == 'E') {
            return CHAR_EXP;
        } else if (ch == '.') {
            return CHAR_POINT;
        } else if (ch == '+' || ch == '-') {
            return CHAR_SIGN;
        } else {
            return CHAR_ILLEGAL;
        }
    }

    bool isNumber(string s) {
        unordered_map<State, unordered_map<CharType, State>> transfer{
            {
                STATE_INITIAL, {
                    {CHAR_NUMBER, STATE_INTEGER},
                    {CHAR_POINT, STATE_POINT_WITHOUT_INT},
                    {CHAR_SIGN, STATE_INT_SIGN}
                }
            }, {
                STATE_INT_SIGN, {
                    {CHAR_NUMBER, STATE_INTEGER},
                    {CHAR_POINT, STATE_POINT_WITHOUT_INT}
                }
            }, {
                STATE_INTEGER, {
                    {CHAR_NUMBER, STATE_INTEGER},
                    {CHAR_EXP, STATE_EXP},
                    {CHAR_POINT, STATE_POINT}
                }
            }, {
                STATE_POINT, {
                    {CHAR_NUMBER, STATE_FRACTION},
                    {CHAR_EXP, STATE_EXP}
                }
            }, {
                STATE_POINT_WITHOUT_INT, {
                    {CHAR_NUMBER, STATE_FRACTION}
                }
            }, {
                STATE_FRACTION,
                {
                    {CHAR_NUMBER, STATE_FRACTION},
                    {CHAR_EXP, STATE_EXP}
                }
            }, {
                STATE_EXP,
                {
                    {CHAR_NUMBER, STATE_EXP_NUMBER},
                    {CHAR_SIGN, STATE_EXP_SIGN}
                }
            }, {
                STATE_EXP_SIGN, {
                    {CHAR_NUMBER, STATE_EXP_NUMBER}
                }
            }, {
                STATE_EXP_NUMBER, {
                    {CHAR_NUMBER, STATE_EXP_NUMBER}
                }
            }
        };

        int len = s.length();
        State st = STATE_INITIAL;

        for (int i = 0; i < len; i++) {
            CharType typ = toCharType(s[i]);
            if (transfer[st].find(typ) == transfer[st].end()) {
                return false;
            } else {
                st = transfer[st][typ];
            }
        }
        return st == STATE_INTEGER || st == STATE_POINT || st == STATE_FRACTION || st == STATE_EXP_NUMBER || st == STATE_END;
    }
};

```

#### 66 加一

```c++
vector<int> plusOne(vector<int>& digits){
	int n = digits.size();
    int carry  = 1;// 初始进位为1
    for(int i = n -1; i >= 0; --i){
        int sum = digits[i] + carry;
        carry = sum / 10;
        digits[i] = sum % 10;
        
        // 如果当前位没有进位，可以提前结束循环
        if(carry == 0){
            break;
        }
    }
     // 如果最高位有进位，则在最前面插入进位
    if(carry > 0){
        digits.insert(digits.begin(), carry);
    }
}
```

#### 67 二进制求和

```c++
string addBinary(string a, string b){
    int i = a.length() -1;
    int j = b.length() -1;
    string res;
    int carry = 0
    while(i >= 0 ||j >= 0 || carry > 0){
        int sum += carry;
        
        if(i >= 0){
            sum += a[i]-'0';
            --i;
        }
        if(j >= 0){
            sum += b[j]-'0';
            --j;
        }
        res = to_string(sum % 2) + res;
        carry = sum / 2;
    }
    return res;
}
```

#### 68 文本左右对齐 (贪心)

```c++
vector<string> fullJustify(vector<string>&words, int maxWidth){
    int n = words.size();
    int start = 0;
    vector<string>res;
    while(start < n){
        int end = start;
        int lineLength = words[end].size();
        while(end + 1 < n && lineLength + words[end+1].size()+1 <= maxWidth){
            ++end;
            lineLength += words[end]+1;  // +1 是因为每个单词后接一个空格
        }
        
        int totalSpaces  = maxWidth - lineLength;
        int numWords = end -start + 1; //每行有多少个单词
        
        if(numsWords == 1 || end = n-1){
            line += words[start];
            for(int i = start + 1; i <= end; ++i){
                line += " " + word[i];
            }
            line.append(maxWidth - line.length(), ' ');
        }else{
            int minSpaces = totalSpaces/(numWords-1);
            int extraSpaces = totalSpaces % (numWords-1);
            
            line += words[start];
            for(int i = start + 1; i <= end; ++i){
                line.append(minSpaces+1, ' ');
                if(extraSpaces > 0){
                    line +=' ';
                    --extraSpaces;
                }
                line += words[i];
            }
        }
        res.emplace_back(line);
        start = end + 1；
    }
    return res;
}
```



#### 69开方（二分法）

```c++
//二分法
int mySqrt(int x){
    if(x == 0) return x;
    int left = 1, right = x, mid, sqrt;
    while(left <= right){
        mid = left + (right - left) / 2;
        sqrt = x / mid;
        if(sqrt == mid)
            return mid;
        else if (sqrt > mid)
            left = mid + 1;
        else
            right = mid -1;
    }
    return right;
}

//牛顿迭代法
int mySqrt(int x){
    long a = x;
    while(a * a > x)
        a = ( a + x / a ) /2;
    return a;
}
```

#### 70爬楼梯(动态规划)

```c++
/*
这是一个经典的动态规划问题，可以使用动态规划的思想来解决。我们可以定义一个数组 dp，其中 dp[i] 表示爬到第 i 阶台阶的方法总数。根据题目要求，爬到第 i 阶台阶的方法总数只取决于爬到第 i-1 阶和第 i-2 阶的方法总数。因此，我们可以得到状态转移方程：

dp[i] = dp[i-1] + dp[i-2]
*/
int climbStairs(int n){
    if(n <= 1){
        return 1;
    }
    vector<int>dp(n+1);
    dp[0] = 1;
    dp[1] = 1;
    for(int i = 2; i <= n; ++i){
        dp[i] = dp[i-1] + dp[i-2];
    }
    return dp[n];
}
```

#### 71 简化路径

```c++
/*
算法思路：

首先将路径以斜杠 '/' 分割为各个部分，存储在一个栈中。
遍历路径的每个部分：
如果是当前目录 "."，则不做任何操作；
如果是上级目录 ".."，则弹出栈顶元素；
否则，将部分入栈。
最后，栈中剩余的元素就是简化后的路径的各个部分。将它们拼接起来，加上斜杠 '/'，得到最终的简化路径。
*/

string simplifyPath(string path){
    vector<string> dirs;
    stringstream ss(path) //它可以方便地对字符串进行输入和输出操作。它的一个主要作用是将字符串分割为多个部分或将多个部分拼接为字符串。
    string dir;
    while(getline(ss, dir, '/')){
        if(dir.empty()||dir == "."){
            continue;
        }else if(dir == ".."){
            if(!dirs.empty()){
                dirs.pop_back();
            }
        }else{
            dirs.push_back(dir);
        }
    }
    string simplifiedPath;
    for(const string& dir : dirs){
        simplifiedPath += "/"+dir;
    }
    return simplifiedPath.empty() ? "/" :simplifiedPath;
}
```

#### 72 编辑距离(动态规划)

```c++
/*
动态规划是解决编辑距离问题的常用方法，它可以有效地将问题拆解为更小的子问题，并利用子问题的解构建出整体的解。对于字符串操作类的问题，动态规划通常是一种高效且可行的解决方案。

在编辑距离问题中，我们需要计算将一个字符串转换为另一个字符串所需的最小操作数（插入、删除、替换操作）。动态规划的思想是，利用一个二维数组（dp数组）来保存从一个子串到另一个子串的编辑距离。

具体而言，对于字符串word1和word2，我们定义dp[i][j]表示将word1的前i个字符转换为word2的前j个字符所需的最小操作数。状态转移方程如下：

如果word1[i]等于word2[j]，则dp[i][j]等于dp[i-1][j-1]，因为不需要进行操作。
如果word1[i]不等于word2[j]，则可以进行三种操作：
插入操作：将word1的前i个字符转换为word2的前j-1个字符，然后在末尾插入word2[j]，此时dp[i][j]等于dp[i][j-1]+1。
删除操作：将word1的前i-1个字符转换为word2的前j个字符，然后删除word1[i]，此时dp[i][j]等于dp[i-1][j]+1。
替换操作：将word1的前i-1个字符转换为word2的前j-1个字符，然后将word1[i]替换为word2[j]，此时dp[i][j]等于dp[i-1][j-1]+1。
最终的答案即为dp[m][n]，其中m和n分别为word1和word2的长度。
*/


int minDistance(string word1, string word2){
    int m = word1.length(); //string 用a.length(), vector 用 a.size()
    int n = word2.length();
    
    vector<vector<int>>dp(m+1, vector<int>(n+1));
    
    //初始化dp数组
    for(int i = 0; i <= m; ++i){
        dp[i][0] = i;
    }
    for(int j = 0; i <= n; ++i){
        dp[0][j] = j;
    }
    
    for(int i = 1; i <= m; ++i){
        for(int j = 1; j <= n; ++i){
            if(word1[i-1] == word2[j-1]){
                dp[i][j] = dp[i-1][j-1];
            }else{
                dp[i][j] = min({dp[i-1][j], dp[i][j-1], dp[i-1][j-1]}) + 1;
            }
        }
    }
    return dp[m][n];
}
```

#### 73 矩阵置零

```c++
/*
是首先用两个标志变量firstRowZero和firstColZero记录第一行和第一列是否包含0。然后使用第一行和第一列来标记矩阵中的0元素，将0元素所在的行和列的第一个元素置为0。接着根据标记，将除第一行和第一列之外的行和列置为0。最后根据firstRowZero和firstColZero来决定是否将第一行和第一列置为0。
*/

void setZeroes(vector<vector<int>>&matrix){
    int m = matrix.size(); //行数
    int n = matrix[0].size(); //列数
    
    bool firstRowZero = false;
    bool firstColZero = false;
    
    //检查第一行是否为0
    for(int j = 0; j < n; ++j){
        if(!matrix[0][j]){
            firstRowZero = true;
            break;
        }
    }
    
    //检查第一列是否为0
    for(int i = 0; i < m; ++i){
        if(!matrix[i][0]){
            firstColZero = true;
            break;
        }
    }
    
    //用第一行和第一列的数据来标记所在行列中为0的数
    for(int i = 1; i < m; ++i){
        for(int j = 1; j < n; ++j){
            if(!matrix[i][j]){
                matrix[i][0] = 0;
                matrix[0][j] = 0;
            }
        }
    }
    
    //检查第一列中为0的行，并将哪行标记为0
    for(int i = 1; i < m; ++i){
        if(!matrix[i][0]){
            for(int j = 1; j < n; ++j){
                matirx[i][j] = 0;
            }
        }
    }
    
    //检查第一行中为0的列，并将那列置为0
    for(int j = 1; j < n; ++j){
        if(!matrix[0][j]){
            for(int i = 1; i < m; ++i){
                matrix[i][j] = 0;
            }
        }
    }
    
    //检查第一行和第一列的标志，如为true 则将其置0
    if(firstRowZero){
        for(int j = 0; j < n; ++j){
            matrix[0][j] = 0;
        }
    }
    
    if(firstColZero){
        for(int i = 0; i < m; ++i){
            matrix[i][0] = 0;
        }
    }
}
```

#### 74 搜索二维矩阵

```c++
bool searchMatrix(vector<vector<int>>&matrix, int target){
    int m = matrix.size();
    int n = matrix[0].size();
    
   	int i = 0, j = n-1; //从右上角开始比较
    while(i < m && j >= 0){
        if(matrix[i][j] > target){
            --j;
        }else if(matrix[i][j] < target){
            ++i;
        }else{
            return true;
        }
    }
    return false;
}
```

#### 75  颜色分类(三指针)

```c++
/*
left指向已排序的0的右边界，right指向已排序的2的左边界，curr用于遍历数组。初始时，left和curr指向数组的起始位置，right指向数组的末尾。
*/

void setColors(vector<int>& nums){
    int left = 0, right = nums.size() -1, curr = 0;
    while(curr <= right){
        if(nums[curr] == 0){
            swap(nums[curr++], nums[left++]);
        }else if(nums[curr] == 2){
            swap(nums[curr], nums[right--]);
        }else{
            ++curr;
        }
    }
}
```



#### 76最小覆盖子串（滑动窗口-双指针）

```c++
string minWindow(string s, string t){
    //先定义两个数组来记录 t中的元素
    vector<int>chars(128,0); //ascii码表 就是 0-127
    vector<bool>flag(128,false);
    
    //记录串t中的元素
    for(int i = 0; i < t.size(); ++t){
        flag[t[i]] = true;
        ++chars[t[i]];
    }
    
    int cnt = 0, l = 0, min_l = 0,minSize = s.size() + 1;
    for(int r = 0; r < s.size(); ++r){
        if(flag[s[r]]){
            if(--chars[s[r]]>=0)
                ++cnt;
        }
        //在已满足条件的情况下，右移左边界
        while(cnt == t.size()){
            if(r-l+1 < minSize){
                min_l = l;
                minSize = r - l + 1;
            }
            if(flag[s[l]] && ++chars[s[l]] > 0)
                --cnt;
            ++l;
        }
    }
    return minSize > s.size() ? "" : s.substr(min_l, minSize);
}
```

#### 77 组合（回溯）

```c++
vector<vector<int>> combine(int n, int k){
    vector<vector<int>>res;
    vector<int>tmp;
    backtrack(n,k,1,tmp,res);
    return res;
}

void backtrack(int n, intk, int start, vector<int>&tmp, vector<vector<int>>&res){
    if(k == 0){
        res.emplace_back(tmp);
        return;
    }
    for(int i = start; i <= n; ++i){
        tmp.push_back(i);
        backtrack(n, k-1, i+1, tmp, res);
        tmp.pop_back();
    }
}
```

#### 78 子集（回溯）

```c++
/*
该解法使用回溯法来生成所有可能的子集。在每个位置上，我们可以选择将当前数字加入子集，或者不加入子集。通过递归实现这个选择过程，最终得到所有可能的子集。

在主函数中，我们初始化一个空的子集subset，然后调用backtrack函数进行递归求解。最后返回得到的所有子集即为结果。
*/

vector<vector<int>> subsets(vector<int>& nums){
    vector<vector<int>>res;
    vector<int>subset;
    backtrack(nums, 0, res, subset);
    return res;
}
void backtrack(vector<int>& nums, int start, vector<vector<int>& res, vector<int>&subset){
    res.emplace_back(temp);
    for(int i = start; i < nums.size(); ++i){
        temp.push_back(nums[i]);
        backtrack(nums, i+1, res, temp);
        subset.pop_back();
    }
}
```

#### 79 单词搜索（dfs 回溯）

```c++
bool exist(vector<vector<char>&board, string word){
    if(board.empty() || board[0].empty()){
        return false;
    }
    int m = board.size();
    int n = board[0].size();
    
    for(int i = 0; i < m; ++i){
        for(int j = 0; j < n; ++j){
            if(dfs(board, word,i , j, 0)){
                return true;
            }
        }
    }
    return false;
}

bool dfs(vector<vector<char>>&board, string word, int row, int col, int pos){
    if(pos == word.size()){
        return true;
    }
    int m = board.size();
    int n = board[0].size()
    if(row < 0 || row >= m || col < 0 || col >= n || board[row][col] != word[pos]){
        return false;
    }
    char temp = board[row][col];
    board[row][col] = '#'; //标记已访问
    bool found = dfs(board, word, row + 1, col, pos+1)||
        		 dfs(board, word, row - 1, col, pos+1)||
    			 dfs(board, word, row, col + 1, pos+1)||
        		 dfs(board, word, row, col - 1, pos+1);
    
    board[row][col] = temp;
      
    return found;
    
}
```

#### 80 删除有序数组的重复项II

```c++
/*
使用两个指针 i 和 j 遍历数组。指针 i 指向当前要保留的元素的位置，指针 j 用于遍历数组。同时使用一个变量 count 记录当前元素的重复次数。当 count 小于等于 2 时，表示当前元素重复次数符合要求，将其保留，同时更新 i 的位置。当 count 大于 2 时，表示当前元素重复次数超过了 2 次，不保留，继续遍历。最后返回 i + 1，即为去重后的数组长度。
*/
int removeDuplicates(vector<int>&nums){
    int n = nums.size();
    if (n <= 2){
        return n;
    }
    int i = 0;
    int count = 1;
    
    for(int j = 1; j < n; ++j){
        if(nums[j] == nums[i]){
            ++count;
        }else{
            count = 1;
        }
        
        if(count <= 2){
            nums[++i] = nums[j];
        }
    }
    return i + 1;
}
```



#### 81 搜索旋转排序数组 II

```c++
bool search(vector<int>& nums, int target){
    int start = 0, end = nums.size() -1;
    while(start <= end){
        int mid = (start + end) >>1;
        if(nums[mid] == target)
            return true;
       	if(nums[start] == nums[mid]){
            //无法判断那个区间是增序
            ++start;
        }else if(nums[mid] <= nums[end]){
            //右区间是增序
            if(target > nums[mid] && target <= nums[end]){
                start = mid + 1;
            }else{
                end = mid - 1;
            }
        }else{
            //左区间是增序
            if(target < nums[mid] && target >= nums[start])
                end = mid -1;
            else
                start = mid + 1;
        }
    }
    return false;
}
```

#### 82 删除排序链表中的重复元素II（双指针）

```c++

/*
其中 prev 指针指向前一个非重复的节点，curr 指针用于遍历链表。

在遍历过程中，首先检查当前节点是否重复。如果重复，通过循环找到下一个不重复的节点，并将 prev 的 next 指针指向该节点，实现删除重复节点的操作。如果当前节点不重复，则将 prev 指针移动到下一个位置。

最后返回虚拟头节点 dummy->next，即为去除重复节点后的链表。
*/
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
 ListNode* deleteDuplicates(ListNode* head){
     if(!head || !head->next){
         return head;
     }
     ListNode* dummy = new ListNode(0);
     ListNode* prev = dummy;
     ListNode* curr = head;
     
     while(curr){
         if(curr->next && curr->val == curr->next->val){
             while(curr->next && curr->val == curr->next->val){
                 curr = curr->next;
             }
             prev = curr->next;
         }else{
             prev = prev->next;
         }
         curr = curr->next;
     }
     return dummy->next;
 }
```

#### 83 删除排序链表中的重复元素

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */

ListNode* deleteDuplicates(ListNode* head){
    ListNode* curr = head, ListNode* prev = head;
    while(curr){
        if(curr->next && curr->val == curr->next->val){
            while(curr->next && curr->val == curr->next->val){
                curr = curr->next;
            }
            prev->next = curr->next;
        }else{
            prev = prev->next;
            curr = curr->next;
        }
    }
    return head;
}
```

#### 84 柱状图中最大的矩形(用栈)

```c++
/*
遍历每个柱子的高度，并将其索引存储在栈中。栈中的柱子保持递增的高度顺序。当我们遇到一个高度小于栈顶柱子的柱子时，我们可以确定栈顶柱子的右边界，计算以栈顶柱子高度为高的矩形的面积，并更新最大面积。然后，我们继续处理新的栈顶柱子，直到栈为空或者遇到一个不小于当前柱子高度的柱子。最后，我们将当前柱子的索引入栈。重复这个过程直到遍历完所有的柱子。

*/
int largestRectangleArea(vector<int>& heights){
    int n = heights.size();
    int maxArea = 0;
    stack<int>st;
    
    for(int i = 0; i <= n; ++i){
        while(!st.empty() && (i == n || heights[i] < height[st.top()])){
            int h = height[st.top()];
            st.pop();
            int w = st.empty()? i : i - st.top()-1;
            maxArea = max(maxArea, h * w);
        }
 		st.push(i);
    }
    return maxArea;
}
```

#### 85 最大矩形

```c++
/*
使用了动态规划和单调栈。首先，我们将二维矩阵转换为一个由每一行的高度组成的一维数组。然后，对于每一行，我们将其看作是一个柱状图，使用上一题中的解法来计算以当前行为底的最大矩形面积。最后，我们取所有行中最大的矩形面积作为结果。

该解法的时间复杂度为 O(m*n)，其中 m 是矩阵的行数，n 是矩阵的列数。我们需要遍历每个元素来计算每一行的高度，并对每一行应用最大矩形面积的算法。
*/
int maximalRectangle(vector<vector<char>>& matrix){
    int m = matrix.size();
    int n = matrix[0].size();
    
    int maxArea = 0;
    vector<int>heights(n, 0);
    for(int i = 0; i < m; ++i){
        for(int j = 0; j < n; ++j){
            if(matrix[i][j] == '1'){
                heights[j] += 1;
            }else{
                heights[j] = 0;
            }
        }
        maxArea = max(maxArea, maxArea(largestRectangleArea(heights))); //largestRectangleArea(heights) 参考上一题
    }
    return maxArea;
}
```

#### 86 分隔链表

```c++
/*
这个解法使用了两个虚拟头节点 smallerHead 和 greaterHead，分别用于存放小于 x 的节点和大于等于 x 的节点。遍历链表，将小于 x 的节点链接到 smallerTail 后面，将大于等于 x 的节点链接到 greaterTail 后面。最后将两个部分连接起来并返回小于 x 的节点的头部。
*/
ListNode* partition(ListNode* head, int x){
    ListNode* smallerHead = new ListNode(0);
    ListNode* smallerTail = smallerHead;
    ListNode* greaterHead = new ListNode(0);
    ListNode* greaterTail = greaterHead;
    ListNode* curr = head;
    
    if(!curr){
        return nullptr;
    }
    while(curr){
        if(curr->val < x){
            smallerTail->next = curr;
            smallerTail = smallerTail->next;
        }else{
            greaterTail->next = curr;
            greaterTail = greaterTail->next;
        }
        curr = curr->next;
    }
    greaterTail->next = nullptr;
    smallerTail->next = greaterHead->next;
    delete greaterHead;
    return smallerHead->next;
}
```

#### 87 扰乱字符（动态规划）

```c++
/*
这个解法利用动态规划来求解。具体思路是使用三维数组 dp 来保存状态转移信息，其中 dp[i][j][k] 表示 s1 从索引 i 开始长度为 k 的子串是否能够通过循环移动得到 s2 从索引 j 开始长度为 k 的子串。通过遍历字符串的长度，枚举划分点位置，然后分别考虑交换和不交换的情况进行状态转移。最终返回 dp[0][0][n]，即整个字符串是否能够满足条件。
*/
bool isScramble(string s1, string s2) {
    int n = s1.length();
    
    // 创建三维数组 dp，用于保存状态转移信息
    // dp[i][j][k] 表示 s1 从 i 开始长度为 k 的子串是否能够通过循环移动得到 s2 从 j 开始长度为 k 的子串
    vector<vector<vector<bool>>> dp(n, vector<vector<bool>>(n, vector<bool>(n + 1, false)));
    
    // 遍历字符串长度为 1 的情况，初始化 dp 数组
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            dp[i][j][1] = (s1[i] == s2[j]);
        }
    }
    
    // 遍历字符串长度大于等于 2 的情况
    for (int len = 2; len <= n; len++) {
        for (int i = 0; i <= n - len; i++) {
            for (int j = 0; j <= n - len; j++) {
                // 划分点位置枚举
                for (int k = 1; k < len; k++) {
                    // 不交换的情况
                    if (dp[i][j][k] && dp[i + k][j + k][len - k]) {
                        dp[i][j][len] = true;
                        break;
                    }
                    // 交换的情况
                    if (dp[i][j + len - k][k] && dp[i + k][j][len - k]) {
                        dp[i][j][len] = true;
                        break;
                    }
                }
            }
        }
    }
    
    return dp[0][0][n];
    }
```

#### 88.归并两个数组

```c++
//从后往前，比较牛皮
void merge(vector<int>&nums1, vector<int>&nums2)
{
    int m = nums1.size(),n = nums2.size();
    int pos = m-- + n-- -1; // =m+n-1
    while(m >= 0 && n >= 0)
    {
        nums1[pos--] = nums1[m] > nums2[n] ? nums1[m--]:nums[n--];
    }
    while(n >= 0)
    {
        nums1[pos--] = nums2[n--];
    }
}

void merge(vector<int>& nums1, int m, vector<int>& nums2, int n) {
        int i = m-1, j = n-1, k= m+n-1;
        while(i >= 0 && j>=0){
            if(nums1[i] < nums2[j]){
                nums1[k--] = nums2[j--];
            }else{
                nums1[k--]  = nums1[i--];
            }
        }

        while(j >= 0){
            nums1[k--] = nums2[j--];
        }
    }
```

#### 89 格雷编码

```c++
/*
这段代码使用了异或运算符 ^ 来生成格雷编码。在格雷编码中，相邻的两个数只有一位不同。通过循环遍历从 0 到 2^n - 1 的数，使用异或运算生成格雷编码，并将结果添加到结果数组中。最后返回结果数组即可。
*/
vector<int> grayCode(int n) {
    vector<int> result;
    int size = 1 << n; // 计算结果的总大小为 2^n
    for (int i = 0; i < size; i++) {
        int num = i ^ (i >> 1); // 使用异或操作生成格雷编码
        result.push_back(num);
    }
    return result;
}
```



#### 90 子集II(回溯)

```c++
vector<vector<int>> sunsetWithDup(vector<int>&nums){
    vector<vector<int>>res;
    sort(nums.begin(),nums.end());
    vector<int>subset;
    backtrack(nums, 0, subset, res);
    return res;
}

void backtrack(vector<int>&nums, int start, vector<int>& subset, vector<vector<int>>& res){
    res.emplace_back(subset);
    
    for(int i = start; i < nums.size(); ++i){
        if(i > start && nums[i] == nums[i-1]) continue;
        subset.push_back(nums[i]);
        backtrack(nums,i+1, subset, res);
        subset.pop_back(nums[i]);
    }
}
```

#### 91 解码方法（动态规划）

```c++
/*
这段代码使用动态规划来计算解码方法数。遍历字符串 s 的每个字符，根据动态规划的转移方程更新 dp 数组的值。最后返回 dp[n]，即为所求的解码方法数。
*/
int numDecodings(string s){
    int n = s.length();
    vector<int>dp(n+1, 0);
    dp[0]= 1;
    
    for(int i = 1; i <= n; ++i){
        if(s[i-1] != '0'){
            dp[i] += dp[i-1];
        }
        
        if(i >= 2){
            int sum = (s[i-2]-'0')*10 + (s[i-1]-'0');
            if(sum >= 10 && sum <= 26){
                dp[i] += dp[i-2];
            }
        }
        return dp[n];
    }
}
```

#### 92 反转链表II

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */

ListNode* reverserBetween(ListNode* head, int left, int right){
    ListNode* dummy = new ListNode(0);
    dummy->next = head;
    ListNode* prev = dummy;
    
    for(int i = 1; i < left; ++i){
        prev = prev->next;
    }
    ListNode* curr = prev->next;
    ListNode* next;
    
    for(int i = left;i < right; ++i){
        next = curr->next;
        curr->next = next->next;
        next->next = prev->next;
        prev->next = next;
    }
    return dummy->next;
}
```

#### 93 复原IP地址（回溯）

```c++
/*
函数restoreIpAddresses是入口函数，其中调用了backtrack函数来进行递归回溯操作。在每一步中，通过截取长度为1到3的子串，并进行合法性判断，来构建符合要求的IP地址。最终，将所有合法的IP地址保存在结果向量res中返回。
*/

vector<string> restoreIpAddress(string s){
    vector<string>res;
    string ip;
    backtrack(s, res, ip, 0, 0);
    return res;
}

void backtrack(string& S, vector<string>& res, string& ip, int start, int count){
    //边界条件：已经形成4段IP地址，并且遍历完整个字符串
    if(count== 4 && start == s.length()){
        res.emplace_back(ip);
        return;
    }
    //边界条件：已经形成4段IP地址，但字符串还未遍历完
    if(count==4 || start == s.length()){
        return;
    }
    
    for(int i = 1; i <= 3; ++i){
        //截取长度为i的子串
        if(start + i > s.length()){
            break;
        }
        
        string segment = s.substr(start, i);//start 是起始位置， i是长度
        
        // 排除不符合IP地址规范的情况
        if((segmrnt[0] == '0' && segment.length() > 1) || stoi(segment) > 255){
            continue;
        }
        
        //在形成IP地址的过程中添加分隔符
        string newIP = ip + segment;
        
        if(count < 3){
            newIP += '.';
        }
        //递归处理下一段
        backtrack(s, res, newIP, start+i, count+1);
    }
}
```

#### 94 二叉树的中序遍历

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */

//迭代
vector<int> inorderTraversal(TreeNode* root){
    vector<int> res;
    strack<TreeNode*>st;
    TreeNode* curr = root;
    while(!st.empty() || curr){
        while(curr){
            st.push(curr);
            curr = curr->left;
        }
        curr = st.top();
        st.pop();
        res.push_back(curr->val);
        curr = curr->right;
    }
    return res;
}
 

//递归
vector<int> inorderTraversal(TreeNode* root){
    vector<int>res;
    inorder(root, res);
    return res;
}
void inorder(TreeNode* root, vector<int>& res){
    if(!root){
        return;
    }
    inorder(root->left, res);
    res.push_back(root->val);
    inorder(root->right, res);
}
```

#### 95 不同的二叉搜索树II

```c++
/*
这个解法使用递归来生成二叉搜索树。首先，确定一个根节点的值（在范围 [start, end] 内选择一个数 i），然后递归生成左子树和右子树，将它们与根节点组合成所有可能的二叉搜索树。

在递归的过程中，如果 start > end，则说明当前子树为空，将 nullptr 加入结果向量中。

最后，返回生成的所有二叉搜索树的根节点组成的向量。
*/
vector<TreeNode*>generateTrees(int n){
    if(n == 0){
        return {};
    }
    return generateTreesHelper(1, n);
}

vector<TreeNode*>generateTreesHelper(int satrt, int end){
    vector<TreeNode*>res;
    if(start > end){
        res.push_back(nullptr);
        return res;
    }
    for(int i = start; i <= end; ++i){
        vector<TreeNode*> leftSubtrees = generateTreesHelper(start, i-1);
        vector<TreeNode*> rightSubtrees = generateTreesHelper(i+1, end);
        
        for(auto left : leftSubtrees){
            for(auto right: rightSubtrees){
                TreeNode* root = new TreeNode(i);
                root->left = left;
                root->right = right;
                res.emplace_back(root);
            }
        }
    }
    return res;
}
```

#### 96 不同的二叉搜索树(动态规划)

```c++
/*
这个解法使用动态规划来计算二叉搜索树的数量。定义一个长度为 n+1 的数组 dp，其中 dp[i] 表示以 1 ... i 为节点组成的二叉搜索树的数量。

初始时，dp[0] 和 dp[1] 的值都为 1，因为没有节点或只有一个节点时只有一种二叉搜索树。

然后，对于每个 i（2 <= i <= n），计算 dp[i] 的值。遍历从 1 到 i 的每个 j，将 j 作为根节点，左子树的节点范围是 1 ... j-1，右子树的节点范围是 j+1 ... i，将左子树和右子树的数量相乘并累加到 dp[i] 中。

最后，返回 dp[n]，即以 1 ... n 为节点组成的二叉搜索树的数量。
*/
int numtrees(int n){
    vector<int>dp(n+1, 0);
    dp[0] =1;
    dp[1] = 1;
    
    for(int i = 2; i <= n; ++i){
        for(int j = 1; i <= i; ++j){
            dp[i] += dp[j-1] *dp[i-j];
        }
    }
    return dp[n];
}
```

#### 97 交错字符串（动态规划）

```c++

/*
这里使用动态规划来解决字符串交错问题。定义一个二维数组 dp，其中 dp[i][j] 表示 s1 的前 i 个字符和 s2 的前 j 个字符是否能够交错形成 s3 的前 i+j 个字符。

初始化时，将 dp[0][0] 设为 true，表示两个空字符串可以交错形成空字符串。

然后，通过填表的方式计算 dp 的其他元素。对于每个位置 (i, j)，如果 s1 的第 i 个字符等于 s3 的第 i+j 个字符，则可以从上方的状态 dp[i-1][j] 转移得到，即 dp[i][j] = dp[i-1][j]。同理，如果 s2 的第 j 个字符等于 s3 的第 i+j 个字符，则可以从左方的状态 dp[i][j-1] 转移得到，即 dp[i][j] = dp[i][j-1]。

最终，返回 dp[m][n]，即判断 s1 的所有字符和 s2 的所有字符是否能够交错形成 s3 的所有字符。如果能够交错形成，则返回 true，否则返回 false。
*/
bool isINterleave(string s1, string s2, string s3){
    int m = s1.size();
    int n = s2.size();
    int k = s3.size();
    if(k != m+n){
        return false;
    }
    vector<vector<bool>>dp(m+1, vector<bool>(n+1, false));
    dp[0][0] = true;
    for(int i = 0; i <= m; ++i){
        for(int j = 0; j <= n; ++j){
            if(i > 0 && s1[i-1] == s3[i+j-1]){
                dp[i][j] = dp[i][j] || dp[i-1][j];
            }
            if(j > 0 && s2[j-1] == s3[i+j-1]){
                dp[i][j] = dp[i][j] || dp[i][j-1];
            }
        }
    }
    return dp[m][n];
}
```



#### 98 验证二叉搜索树(递归)

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
bool isValidBST(TreeNode* root){
    return isValidBSTHelper(root, nullptr, nullptr);
}

bool isValidBSTHelper(TreeNode* node, TreeNode* minNode, TreeNode* maxNode){
    if(!node){
        return true;
    }
    if((minNode && node->val <= minNode->val) || (maxNode && node->val >= maxNode->val)){
        return false;
    }
    return isValidBSTHelper(node->left, minNode, node) && isValidBSTHelper(node->right, node, maxNode);
}
```

#### 99 恢复二叉搜索树

```c++
TreeNode* prev = nullptr;
TreeNode* x = nullptr;
TreeNode* y = nullptr;

void recoverTree(TreeNode* root){
    inorderTraversal(root);
    swap(x->val, y->val);
}
void inorderTraversal(TreeNode* root){
    if(!root){
        return;
    }
    inorderTraversal(root->left);
    if(prev && prev->val > root->val){
        if(!x){
            x = prev;
        }
        y = root;
    }
    prev = root;
    inorderTraversal(root->right);
}
```

#### 100 相同的树

```c++
bool isSameTree(TreeNode* p, TreeNode* q){
    if(!p && !q){
        return true;
    }
    if(!p || !q){
        return false;
    }
    if(p->val != q->val){
        return false;
    }
    return isSameTree(p->left, q->left) &&  isSameTree(p->right, q->right);
}
```

#### 101 对称二叉树

```c++
bool isSymmetric(TreeNode* root){
    if(!root){
        return true;
    }
    return isSymmetricHelper(TreeNode* root->left, TreeNode* root->right);
}

bool isSymmetricHelper(TreeNode* p, TreeNode* q){
    if(!p && !q){
        return true;
    }
    if(!p || !q){
        return false;
    }
    if(p->val != q->val){
        return false;
    }
    return isSymmetricHelper(p->left, q->right) && isSymmetricHelper(p_>right, q->left);
}
```

#### 102 二叉树的层序遍历

```c++
vector<vector<int>> levelOrder(TreeNode* root){
 	vector<vector<int>>res;
    if(!root){
        return res;
    }
    queue<TreeNode*>q;
    q.push(root);
    while(!q.empty()){
        int levelSize = q.size();
        vector<int>levelNodes;
        
        for(int i = 0; i < levelSize; ++i){
            
        }
    }
}
```

#### 103 二叉树锯齿形层序遍历

```c++
vector<vector<int>> zipzaglevelOrder(TreeNode* root){
    int flag = 1;
    vector<vector<int>>res;
    if(!root){
        return res;
    }
    queue<TreeNode*>q;
    q.push(root);
    while(!q.empty()){
        int levelSize = q.size();
        vector<int>levelNodes(levelSize);
        
        for(int i = 0; i < leveSize; ++i){
            TreeNode* node = q.front();
            q.pop();
            int index = flag > 0 ? i : leveSize -i -1;
            levelNodes[index] = node->val;
            if(node->left){
                q.push(node->left);
            }
            if(node->right){
                q.push(node->right);
            }
        }
        res.emplace_back(levelNodes);
        flag = -flag;
    }
    return res;
}
```

#### 104 二叉树的最大深度

```c++
int maxDepth(TreeNode* root){
    if(!root){
        return 0;
    }
    int leftDepth = maxDepth(root->left);
    int rightDepth= maxDepth(root->right);
    
    return max(leftDepth, rightDepth) + 1;
}
```

#### 105 从前序与中序遍历序列构造二叉树

```c++
TreeNode* buildTree(vector<int>&preorder, vecot<int>&inorder){
    //使用哈希表存储中序遍历的值与索引的映射关系，提高查找效率
    unordered_map<int, int>inorderMap;
    for(int i = 0;i < inorder.size(); ++i){
        inorderMap[inorder[i]] = i;
    }
    return buildTreeHelper(preorder, 0, preorder.size()-1, inorder, 0, inorder.size()-1, inorderMap);
}

TreeNode* buildTreeHelper(vector<int>& preorder, int preStart, int preEnd,
                         vector<int>& inorder, int inStart, int inEnd,
                         unordered_map<int,int>& inorderMap){
    
    //递归终止条件
    if(preStart > PreEnd || inStart > inEnd){
        return nullptr;
    }
    //根据前序遍历确定根节点
    int rootVal = preorder[preStart];
    TreeNode * root = new TreeNode(rootVal);
    
    //在中序遍历中找到根节点的索引
    int rootIndex = inorderMap[rootVal];
    
    //计算左子树的长度
    int leftSubtreeLength = rootIndex - inStart;
    
    //构造左子树和右子树
    root->left = buildTreeHelper(preorder, preStart+1, preStart+leftSubtreeLength,
                                inorder, inStart, rootIndex -1,inorderMap);
    root->right = buildTreeHelper(preorder, preStart+leftSubtreeLength+1, preEnd,
                                inorder, rootIndex +1, inEnd,inorderMap);
    return root;
}
```

#### 106 从中序与后序遍历构造二叉树

```c++
TreeNode* buildTree(vector<int>&inorder, vector<int>&postorder){
 	unordered_map<int>inorderMap;
    
    for(int i = 0; i < inorder.size(); ++i){
        inorderMap[inorder[i]] = i;
    }
    
    return buildTreeHelper(inorder, 0, inorder.size()-1, postorder, 0, postorder.size()-1, inorderMap);
}

TreeNode* buildTreeHelper(vector<int>& inorder, int inStart, int inEnd,
                         vector<int>& postorder, int poStart, int poEnd, 
                         unordered_map<int, int>& inorderMap){
    //递归的边界条件
    if(inSatrt > inEnd || poStart > poEnd){
        return nullptr;
    }
    
    int rootVal = postorder[poEnd];
    TreeNode* root = new TreeNode(rootval);
    
    int index = inorderMap[rootVal];
    int rightLen = inEnd - index;
    
    root->left = buildTreeHelper(inorder, inStart, index-1, 
                                 postorder, poStart, poEnd-rightLen-1,
                                inorderMap);
    root->right = buildTreeHelper(inorder, index+1, inENd,
                                 postorder, poEnd- rightLen, poEnd-1,
                                 inorderMap);
    return root;
}
```

#### 107 二叉树的层序遍历II

```c++
//反转
vector<vector<int>> levelOrderBottom(TreeNode* root){
    vector<vector<int>>res;
    if(!root){
        return res;
    }
    queue<TreeNode*>q;
    q.push(root);
    
    while(!q.empty()){
        int levelSize = q.size();
        vector<int>levelNodes;
        
        for(int i = 0; i < levelSize; ++i){
            TreeNode* node = q.front();
            q.pop();
            
            levelNodes.push_back(node->val);
            
            if(node->left){
                q.push(node->left);
            }
            
            if(node->right){
                q.push(node->right);
            }
        }
        res.emplace_back(levelNodes);
    }
    reverse(res.begin(), res.end());
    return res;
}
```

#### 108 将有序数组转换成二叉搜索树

```c++
TreeNode* sortedArraytoBST(vector<int>& nums){
    int n = nums.size()-1;
    return sortedArraytoBSTHelper(nums, 0, n);
}

TreeNode* sortedArraytoBSTHelper(vector<int>& nums, int start, int end){
    if(start > end){
        return nullptr;
    }
    
    int mid = (start + end) >> 1;  //右移相当于除以2
    TreeNode* root = new TreeNode(nums[mid]);
    
    root->left = sortedArraytoBSTHelper(nums, satrt, mid-1);
    root->right = sortedArraytoBSTHelper(nums, mid+1, end);
    return root;
}
```

#### 109 有序链表装换二叉搜索树

```c++
TreeNode* sortedListtoBST(ListNode* head){
    if(!head) return nullptr;
    if(!head->next) return new TreeNode(head->val);
    ListNode* prev= head;
    ListNode* p =head;
    int count = 0;
    
    //计算节点数
    while(p){
        ++count;
        p = p->next;
    }
    //找到根节点
    p = head;
    int mid = count >> 1;
    while(mid--){
        prev = p;
        p = p->next;
    }
    prev->next = nullptr;
    TreeNode* root = new TreeNode(p->val);
    root->left = sortedListtoBST(head);
    root->right = sortedListtoBST(p->next);
    return root;
}
```

#### 110 平衡二叉树

```c++
bool isBalanced(TreeNode* root){
    return checkHeight(root) != -1
}

int checkHeight(TreeNode* root){
    if(!root){
        return 0;
    }
    int leftHeight = checkHeight(root->left);
    if(leftHeight == -1){
        return -1;
    }
    
    int rightHeight = checkHeight(root->right);
    if(rightHeight == -1){
        return -1;
    }
    
    if(abs(leftHeight - rightHeight) > 1){
        return -1;
    }
    return max(leftHeight, rightHeight) + 1;
}
```

#### 111 二叉树的最小深度

```c++
int minDepth(TreeNode* root){
    if(!root){
        return 0;
    }
    if(!root->left) return minDepth(root->right);
    if(!root->right) return minDepth(root->left);
    
    int leftDepth = minDepth(root->left);
    int rightDepth = minDepth(root->right);
    
    return min(leftDepth, rightDepth)+1;
}
```

#### 112 路径总和

```c++
bool hasPathSum(TreeNode* root, int targetSum){
    if(!root){
        return false;
    }
    if(!root->left && !root->next){
        if(root->val == targetSum){
            return true;
        }
    }
    return hasPathSum(root->left, targetSum - root->val) || hasPathSum(root->right, targetSum - root->val);
}
```

#### 113  路径总和II（dfs 深度优先搜索）

```c++
vector<vector<int>> pathSum(TreeNode* root, int targetNum){
    vector<vector<int>>res;
    vector<int>path;
    dfs(root, targetNum, res, path);
    return res;
}

void dfs(TreeNode* node, int targetNum, vector<vector<int>>&res, vector<int>& path){
	if(!root){
        return;
    }
    path.push_back(node->val);
    if(!root->left && !root->right && node->val == targetNum){
        res.emplace_back(path);
    }
    dfs(node->left, targetNum-node->val, res, path);
    dfs(node->right, target-node->val, res, path);
    path.pop_back();
}
```

#### 114 二叉树展开为链表

```c++
void flatten(TreeNode* root){
	if(!root){
        return;
    }
    TreeNode* dummy = new TreeNode(0);
    TreeNode* prev = dummy;
    flattenHelper(root, prev);
    delete dummy;
}

void flattenHelper(TreeNode* root, TreeNode* prev){
    if(!root){
        return;
    }
    
    prev->right = root;
    prev->left = nullptr;
    prev = root;
    TreeNode* right = root->right;
    flattenHelper(root->left, prev);
    flattenHelper(right, prev);
}
```

#### 115 不同的子序列(动态规划)

```c++
int numDistinct(string s, string t){
	vector<int>chars(128, 0);
    vector<bool>flag)(128, false);
    
    for(char c : t){
        flag[c] = true;
        ++chars[c];
    }
    
    vector<long>dp(t.size() + 1. 0);
    dp[0] = 1;
    
    for(char c : s){
        if(!flag[c]){
            continue;
        }
        
        for(int i = t.size(); i >= 1; --i){
            if(c == t[i-1]){
                dp[i] += dp[i-1];
                if(dp[i] > INT_MAX){
                    dp[i] = INT_MAX;
                }
            }
        }
    }
    return dp[t.size()];
}
```

#### 116 填充每个节点的下一个右侧节点指针（层序遍历）

```c++
//完美二叉树
//层序遍历
Node* connect(Node* root){
    if(!root){
        return nullptr;
    }
    
    queue<Node*>q;
    q.push(root);
    while(!q.empty()){
        int levelSize = q.size();
        Node* prev = nullptr;
        for(int i = 0; i < levelSize; ++i){
            Node* node = q.front();
            q.pop();
            
            if(prev){
                prev->next = node;
            }
            if(node->left){
                q.push(node->left);
            }
            if(node->right){
                q.push(node->right);
            }
            prev = node;
        }
        prev->next = nullptr;
    }
    return root;
}

//递归
Node* connect(Node* root){
    if(!root){
        return nullptr;
    }
    if(root->left){
        root->left->next = root->right;
    }
    if(root->right){
        if(root->next){
            root->right->next = root->next->left;
        }else{
            root->right->next = nullptr;
        }
    }
    connect(root->left);
    connect(root->right);
    return root;
}
```

#### 117 填充每个节点的下一个右侧节点指针II

```c++
//普通二叉树
//层序遍历方法同样可行

//迭代

/*
在这段代码中，我们使用迭代方法来连接每个节点的next指针。我们维护一个指针currLevel，表示当前层级的最左节点。

在外层循环中，我们遍历当前层级的节点。在内层循环中，我们遍历当前层级的节点，并连接其左右子节点的next指针。我们使用prev指针来记录前一个连接的节点，确保连接的顺序是正确的。

在内层循环结束后，我们将currLevel指向下一层级的最左节点，然后继续外层循环，直到遍历完所有层级。

最后，我们返回根节点root。
*/
Node* connect(Node* root){
    if(!root){
        return nullptr;
    }
    
    Node* currLevel = root;
    while(currLevel){
        Node* dummy = new Node();
        Node* prev = dummy;
        
        Node* curr = currLevel;
        while(curr){
            if(curr->left){
                prev->next = curr->left;
                prev = prev->next;
            }
            
            if(curr->right){
                prev->next = curr->right;
                prev = prev->next;
            }
            curr = curr->next;
        }
        currLevel = dummy->next;
        delete dummy;
    }
    return root;
}
```

#### 118 杨辉三角

```c++
vector<vector<int>> generate(int numRows){
    vector<vector<int>>res;
    if(!numRows){
        return res;
    }
    
    res.push_back({1});
    
    for(int i = 1; i < numRows; ++i){
        vector<int>temp(i+1, 1); // 都初始化为1
        for(int j = 1; j < i; ++j){
            temp[j] = res[i-1][j-1] + res[i-1][j];
        }
        res.emplace_back(temp);
    }
    return res;
}
```

#### 119 杨辉三角II（动态规划）

```c++
// 要求生成杨辉三角的第n行
/*
在这段代码中，我们创建了一个大小为rowIndex + 1的向量row，并将其初始化为1，表示第一行。

然后，我们从第三行开始，通过遍历更新每个元素的值。对于第i行的第j个元素，它等于上一行的第j个元素和第j-1个元素之和。我们通过从后向前的循环，更新当前行的元素值。

最终，我们返回生成的第rowIndex行。
*/
vector<int> getRow(int rowIndex){
    vector<int> row(rowIndex+1, 1);
    
    for(int i = 2; i <= rowIndex; ++i){
        for(int j = i-1; j >= 1; --j){
            row[j] += row[j-1];
        }
    }
    return row;
}
```

#### 120 三角形最小路径和（动态规划）

```c++
int minmumTotal(vector<vector<int>>& triangle){
    int n = triganle.size();
    
    //处理特殊情况。当三角形为空是，最下路径和为0
    if(!n){
        return 0;
    }
    //初始化为最后一行，自底向上，保存每一行的最小路径和
    vector<int>dp = triangle[n-1];
    
    for(int i = n-2; i >= 0; --i){ // 从倒数第二行开始，逐行更新dp数组
        for(int j = 0; j <= i; ++j){
            dp[j] = triganle[i][j] + min(dp[j], dp[j+1]);
        }
    }
    //最终dp数组的第一个元素即为最小路径和
    return dp[0];
}
```

#### 121 买卖股票的最佳时机

```c++
int maxProfit(vector<int>& prices){
    int n = prices.size();
    
    if(n <= 1){
        return 0;
    }
    
    int minPrice = prices[0]; // 记住最低股票价格
    int maxPro = 0; // 记录最大利润
    
    for(int i = 1; i < n; ++i){
        if(prices[i] < minPrice){
            minPrice = prices[i]; // 更新最低股票价格
        }else if(prices[i] - minPrice > maxPro){
            maxPro =  prices[i]-minPrice; //更新最大利润
        }
    }
    return maxPro;
}
```



#### 122 买卖股票的最佳时机 II(贪心)

```c++
//只要 差值>0 就 +
int maxProfit(vector<int>&prices)
{
    int res = 0;
    for(int i = 1; i <prices.size();++i)
    {
        res += (prices[i] - prices[i+1]) > 0 ? prices[i] - prices[i+1] : 0;
    }
    return res;
}
```

#### 123 买卖股票的最佳时机 III

```c++
// 最多可以完成两笔交易

int maxProfit(vector<int>& prices){
    int n = prices.size();
     if (n <= 1) {
        return 0;
    }
    
    int buy1 = INT_MAX; // 第一次买入的最低价格
    int buy2 = INT_MAX; // 第二次买入的最低价格
    int sell1 = 0; // 第一次卖出的最大利润
    int sell2 = 0; // 第二次卖出的最大利润
    
    for (int i = 0; i < n; i++) {
        // 第一次买入的最低价格
        buy1 = min(buy1, prices[i]);
        // 第一次卖出的最大利润
        sell1 = max(sell1, prices[i] - buy1);
        // 第二次买入的最低价格
        buy2 = min(buy2, prices[i] - sell1);
        // 第二次卖出的最大利润
        sell2 = max(sell2, prices[i] - buy2);
    }
    
    return sell2;
}
```

#### 124 二叉树的最大路径和

```c++
int maxPathSum(TreeNode* root){
    int maxSum = INT_MIN; //记录最大路径和
    maxPathSumHelper(root, maxSum);
    return maxSum;
}

int maxPathSumHelper(TreeNode* node, int& maxSum){
    if(!node){
        return 0;
    }
    
    int leftSum = max(0, maxPathSumHelper(root->left, maxSum));
    int rightSum = max(0, maxPathSumHelper(root->right, maxSum));
    
    //当前节点作为路径的给二孃2节点时，计算当前路径和
    int currSum = node->val + leftSum + rightSum;
    
    //更新最大路径和
    maxSum = max(maxSum, currSum);
    
    // 返回当前节点作为路径一份不得最大路径和
    return node->val+max(leftSum, rightSum);
    
}
```

#### 125 验证回文串(双指针)

```c++
bool isPalindrome(string s){
    bool isPalindrome(string s) {
        int left = 0; // 左指针
        int right = s.length() - 1; // 右指针
        
        while (left < right) {
            // 跳过非字母和数字字符
            while (left < right && !isalnum(s[left])) {
                left++;
            }
            while (left < right && !isalnum(s[right])) {
                right--;
            }
            
            // 将字符转换为小写字母进行比较
            if (tolower(s[left]) != tolower(s[right])) {
                return false; // 如果不相等，则不是回文字符串
            }
            
            left++;
            right--;
        }
        
        return true; // 左右指针相遇，说明是回文字符串
    }
```

#### 126  单词接龙 II (超难)

```c++
vector<vector<string>>findLadders(string beginWord, string endWord, vector<string>&wordList){
    
    vector<vector<string>>res;
    unordered_set<string>dict = {wordList.begin(),wordList.end()};
    if(!dict.count(endWord)){
        return res;
    }
    
    dict.erase(beginWord);
    
    unordered_map<string, int>steps = {{beginWord, 0}};
    
    unordered_map<string, set<string>>
}
```

#### 127 单词接龙(BFS + dict + queue)

```c++
/*
该算法采用了 BFS 算法，在一个图中寻找从起点 beginWord 到终点 endWord 的最短路径。其中，将 wordList 转换成一个哈希表 dict 以便进行快速查找，然后从起点开始，逐个遍历其每一位上所有可能的变形，查找字典是否包含此变形，如果包含就加入队列，然后在字典中删除此变形以避免重复搜索。

*/
int ladderLength(string beginWord, string endWord, vector<string>&wordList){
    unordered_set<string>dict(wordList.begin(), wordList.end());
    if(!dict.count(endWord)){
        return 0;
    }
    
    queue<string>q{{beginWord}};
    
    while(!q.empty()){
        for(int k = q.size(); k >0; --k){
            string word = q.front();
            q.pop();
            if(word == endWord){
                return res+1;
            }
            for(int i = 0; i < word.size(); ++i){
                char ch = word[i];
                for(int j = 'a'; j <= 'z'; ++j){
                    word[i] = j;
                    if(dict.count(word)){
                        dict.erase(word);
                        q.push(word);
                    }
                }
                word[i] = ch;
            }
        }
        ++res;
    }
    return 0;
}
```

#### 128 最长连续序列（unordered_set）

```c++
int longestConsecutive(vector<int>& nums){
    unordered_set<int>num_set(nums.begin(), nums.end());
    int maxLen = 0;
    
    for(auto num:nums){
        if(!num_set.count(num-1)){
            int curr_num = num;
            int curr_len = 1;
            while(num_set.count(curr_num + 1)){
                ++curr_num;
                ++curr_len;
            }
            maxLen = max(maxLen, curr_len);
        }
    }
    return maxLen;
}
```

#### 129 求根节点到叶子节点数字之和（dfs）

```c++
int sumNumbers(TreeNode* root){
    int sum = 0;
    dfs(root, 0, sum);
    return sum;
}

void dfs(TreeNode* node, int pathSum, int& sum){
    if(!node){
        return;
    }
    pathSum  = pathSum * 10 + node->val;
    
    if(!node->left && !node->right){
        sum += pathSum;
    }
    dfs(node->left, pathSum, sum);
    dfs(node->right, pathSum, sum);
    
    pathSum /= 10;
}
```



#### 130 被围绕的区域（dfs）

```c++
void solve(vector<vector<char>>& board){
    int m = board.size(), n = board[0].size();
    
    //遍历边缘上的'O',并深度优先搜索相邻的'O'
    for(int i = 0; i < m; ++i){
        dfs(board, i, 0); //第一列
        dfs(board, i, n-1);//最后一列
    }
    
    for(int j = 0; j < n; ++j){
        dfs(board, 0, j); // 第一行
        dfs(board, m-1, j); // 最后一行
    }
    
    //将未标记的'O'替换成'X'，已标记的'O' 恢复成'O'
    for(int i = 0; i < m; ++i){
        for(int j = 0; j < n; ++j){
            if(board[i][j] == 'O'){
                board[i][j] = 'X';
            }
            if(board[i][j] == '#'){
                board[i][j] = 'O';
            }
        }
    }
}

void dfs(vector<vector<char>>& board, int x, int y){
    if(x < 0 || x >= board.size() || y < 0 || y >= board[0].size() || board[x][y] != 'O'){
        return;
    }
    board[x][y] = '#'; //标记已遍历过的'O'
    dfs(board, x-1, y);//上
    dfs(board, x+1, y);//下
    dfs(board, x, y-1);//左
    dfs(board, x, y+1);//右

}
```

#### 131 分割回文串(dfs)

```c++
vector<vector<string>> partition(string s){
    vector<vector<string>>res;
    vector<string>path;
    dfs(s, 0, path, res);
    return res;
}

void dfs(string& s, int start, vector<string>& path, vector<vector<string>>& res){
    if(start == s.size()){
        res.emplace_back(path);
        return;
    }
    for(int i = start; i < s.size(); ++i){
        if(isPalindrome(s, start, i)){
            path.emplace_back(s.substr(start, i-start+1));
        	dfs(s, i + 1, path, res);
        	path.pop_back();
        }
    }
}

bool isPalindrome(string& s, int start, int end){
    while(start < end && s[start] == s[end]){
        ++start;
        --end;
    }
    return start >= end;
}
```

#### 132 分割回文串II （返回最小割， 动态规划）

```c++
int minCut(string s){
    int n = s.size();
    vector<int>dp(n+1, 0);
    for(int i = 0; i <= n; ++i){
        dp[i] = i-1;
    }
    
    for(int i = 0; i < n; ++i){
        for(int j = 0; i-j >= 0 && i+j < n && s[i-j] == s[i+j]; ++j){
            dp[i+j+1] = min(dp[i+j+1], dp[i-j]+1);
        }
        for(int j = 1; i-j+1 >=0 && i+j < n && s[i-j+1] == s[i+j];++j){
            dp[i+j+1] = min(dp[i+j+1], dp[i-j+1]+1);
        }
    }
    return dp[n];
}
```

#### 133 克隆图

```c++
/*
// Definition for a Node.
class Node {
public:
    int val;
    vector<Node*> neighbors;
    Node() {
        val = 0;
        neighbors = vector<Node*>();
    }
    Node(int _val) {
        val = _val;
        neighbors = vector<Node*>();
    }
    Node(int _val, vector<Node*> _neighbors) {
        val = _val;
        neighbors = _neighbors;
    }
};
*/
unordered_map<Node*, Node*> visted;
    Node* cloneGraph(Node* node) {
        if(!node){
            return node;
        }

        if(visted.count(node)){
            return visted[node];
        }

        Node* cloneNode = new  Node(node->val);
        visted[node] = cloneNode;
        for(auto neighbor : node->neighbors){
            cloneNode->neighbors.emplace_back(cloneGraph(neighbor));
        }
        return cloneNode;
    }
```

#### 134 加油站

```c++
int canCompleteCircuit(vector<int>& gas, vector<int>& cost){
    int n = gas.size();
    int total = 0;
    int curr = 0;
    int startStation = 0;
    for(int i = 0; i < n;++i){
        total += gas[i] - cost[i];
        curr += gas[i] - cost[i];
        if(curr < 0){
            startStation = i + 1;
            curr = 0;
        }
    }
    return total >= 0 ?  startStation : -1;
}
```



#### 135分发糖果（贪心）

```c++o
int candy(vector<int>& ratings)
{
    int n = ratings.size();
    if(n < 2)
        return n;
    int i = 0;
    vector<int>res(n, 1);
    for(i = 1; i < n; ++i)
    {
        if(ratings[i] > ratings[i-1])
            res[i] = res[i-1] + 1;
    }
    for(i=n-1;i>0;--i)
    {
        if(ratings[i] < ratings[i-1])
            ratings[i-1] = max(ratings[i-1], ratings[i] + 1);
    }
    return accumulate(res.begin(), res.end(),0);
}
```

#### 136 只出现一次的数字(^)

```cpp
int sngleNumber(vector<int>& nums){
    int res = 0;
    for(int i = 0; i < nums.size(); ++i){
        res ^= nums[i];
    }
    return res;
}
```

#### 137 只出现一次的数字II

```cpp
int singleNumber(vector<int>& nums){
    int ones =0, twos = 0;
    for(int num: nums){
        ones = (ones ^ num) & ~twos;
        twos = (twos ^ nums) & ~ones;
    }
    return ones;
}
```

#### 138 复制带随机指针的链表(hash)

```cpp
Node* copyRandomList(Node* head){
    if(!head)
        return nullptr;
    unordred_map<Node*, Node*>copiedMap;
    Node* curr = head;
    while(curr){
        copiedMap[curr] = new Node(curr->val);
        curr = curr->next;
    }
    curr = head;
    while(curr){
        copiedMap[curr]->next = copiedMap[curr->next];
        copiedMap[curr]->random = copiedMap[curr->random];
        curr = curr->next;
    }
    return copiedMap[head];
}
```

#### 139 单词拆分(dp)

```cpp
bool wordBreak(string s, vector<string>& wordDict){
    unordered_set<string>wordSet(wordDict.begin(), wordDict.end());
    int n = s.length();
    vector<bool>dp(n+1, false);
    dp[0] = true;
    
    for(int i = 1; i <= n; ++i){
        for(int j = 0; j < i; ++j){
            if(dp[j] && wordSet.find(s.substr(j, i-j)) != wordSet.end()){
                dp[i] = true;
                break;
            }
        }
    }
    return dp[n];
}
```

#### 140 单词拆分II

```cpp
vector<string> wordBreak(string s, vector<string>& wordDict){
    unordered_set<string> wordSet(wordDict.begin(), wordDict.end());
    unordered_set<int> vaildEnds;
    vector<string>res;
    string curr;
    backtrack(s, res, curr, 0, vaildEnds, wordSet);
    return res;
}

void backtrack(string s, vector<string>& res, string& curr, int st,
               unordered_set<int>& validEnds,
              unordered_set<string>& wordSet){
    if(st = s.length()){
        res.emplace_back(curr);
        return;
    }
    vaildEnds.insert(st);
    for(int i = st+1; I < s.length(); ++i){
        string word = s.substr(st, i-st);
        if(wordSet.find(word) ! = wordSet.end() && validEnds.find(st) != vaildEnds.end()){
            string temp = curr.empty() ? word: curr + " "+ word;
            string prevCurr = curr;

            curr = temp;
            backtrack(s, res, curr, i, validEnds, wordSet);
            curr = prevCurr;
        }
    }
}
```

#### 141 环形链表(快慢指针)

```cpp
bool hasCycle(ListNode* head){
    if(!head || !head->next)
        return false;
    ListNode* fast = head->next;
    ListNode* slow = head;
    while(fast != slow){
        if(!fast || !fast->next)
            return false;
        fast = fast->next->next;
        slow = slow->next;
    }
    return true;
}
```



#### 142 环形链表II (快慢指针-双指针)

```c++
//快慢指针
//无特殊说明，LeetCode采用如下的数据结构表示链表
struct ListNode{
    int val;
    ListNode *next;
    ListNode(int x):val(x), next(nullptr){}
};

ListNode *detectCycle(ListNode *head){
    ListNode *fast = head, *slow = head;
    do{
        if(!fast || !fast->next) return nullptr;
        fast = fast->next->next;
        slow = slow->next;
    }while(fast != slow);
    
    fast = haed;
    while(fast != slow){
        fast = fast->next;
        slow = slow->next;
    }
    return fast;
}

```

#### 143 重排链表

```cpp
void reorderList(ListNode* head){
    if(!head || !head->next){
        return;
    }
    ListNode* slow = head;
    ListNode* fast = head;
    
    while(fast->next && fast->next->next){
        slow = slow->next;
        fast = fast->next->next;
    }
    
    ListNode* secondHalf = slow->next;
    slow->next = nullptr;
    
    secondHalf = reverseList(secondHalf);
    
    ListNode* firstPtr = head;
    LiseNode* secondPtr = secondHalf;
    
    while(secondPtr){
        ListNode* temp1 = firstPtr->next;
        ListNode* temp2 = secondPtr->next;
        
        firstPtr->next = secondptr;
        secondPtr->next = temp1;
        
        firstPtr = temp1;
        secondPtr = temp2;
    }
    
    if(firstPtr)
        firstPtr->next = nullptr;
}

ListNode* reverseList(ListNode* head){
    ListNode* dummy = new ListNode(0);
    ListNode* cur = head;
    ListNode* next;
    dummy->next = nullptr;
    while(cur){
        next = cur->next;
        cur->next = dummy->next;
        dummy->next = cur;
        cur = next;
    }
    ListNode* newHead = dummy->next;
    delete(dummy);
    return newHead;
}
```

#### 144 二叉树的前序遍历

```cpp
vector<int> preorderTraversal(TreeNode* root){
    vector<int>res;
    preorder(root,res);
    return res;
}

void preorder(TreeNode* node, vector<int>& res){
    if(!node)
        return;
    res.push_back(node->val);
    preorder(node->left, res);
    preorder(ndoe->right, res);
}
```

#### 145 二叉树的后序遍历

```cpp
vector<int> postorderTraversal(TreeNode* root){
    vector<int>res;
    postorder(root, res);
    return res;
}

void(TreeNode* node, vector<int>& res){
    if(!node)
        return;
    postorder(node->left, res);
    postorder(node->right, res);
    res.push_back(node->val);
}
```

#### 146 LRU 缓存（List + hash）

```cpp
class LRUCache{
public:
    LRUCache(int cap)
        :capacity(cap){}
    
    int get(int key){
        if(cache.find(key) == cache.end()){
            return -1;  
        }
        
        //将访问的元素移到列表头部
        auto it = cache[key].second;
        lru.erase(it);
        lru.push_front(key);
        cache[key].second = lru.begin();
    }
    
    void put(int key, int value){
        if(cache.find(key) != cache.end())
        {
            cache[key].first = value;
            auto it = cache[key].second;
            lru.erase(it);
            lru.push_front(key);
            cache[key].second = lru.begin();
        }else{
            if(lru.size() >= capacity){
                int lastkey = lru.back();
                lru.pop_back();
                cache.erase(lastkey);
            }
            
            lru.push_front(key);
            cache[key] = {value, lru.begin()};
        }
    }
private:
    int capacity;
    unordered_map<int, pair<int, list<int>:: iterator>> cache;
   	list<int> lru;
};
```

#### 147 对链表进行插入排序

```cpp
ListNode* insertSortList(ListNode* head){
    if(!head || !head->next){
        return head;
    }
    
    ListNode dummy(0);
    dummy.next = head;
    ListNode* curr = head->next;
    ListNode* prev = head;
    
    while(curr){
        if(curr->val >= prev->val){
            curr = curr->next;
            prev = prev->next;
        }else{
            ListNode* nextNode = curr->next;
            ListNode* insertPos = &dummy;
            
            while(insertPos->next->val < curr->val){
                insertPos = InsertPos->next;
            }
            
            curr->next = insertPos->next;
            insertPost->next =  curr;
            
            prev->next = nextNode;
            curr = nextNode;      
        }
    }
    return dummy.next;
}
```

#### 148 排序链表（归并）

```cpp
ListNode* sortList(ListNode* head){
    if(!head || !head->next){
        return head;
    }
    
    ListNode* dummy = new ListNode(0);
    dummy->next = head;
    int len = getLength(head);
    
    for(int step = 1; step < len; step <<= 1){
        ListNode* prev = dummy;
        ListNode* curr = dummy->next;
        
        while(curr){
            ListNode* left = curr;
            ListNode* right = split(left, step);
            
            curr = split(right, step);
            
            prev = merge(right, left, prev);
        }
    }
    ListNode* sortedHead = dummy->next;
    delete dummy;
    return sortedHead;
}

int getLength(ListNode* head){
    int len = 0;
    while(head){
        ++len;
        head = head->next;
    }
    return len;
}

ListNode* split(ListNode* head, int n){
    for(int i = 1; head && i < n; ++i){
        head = head->next;
    }
    if(!head){
        return nullptr;
    }
    ListNode* nextNode = head->next;
    head-next = nullptr;
    return nextNode;
}

ListNode* merge(ListNode* l1, ListNode* l2, ListNode* prev){
    ListNode* curr = prev;
    while(l1 && l2){
        if(l1->val < l2->val){
            curr->next = l1;
            l1 = l1->next;
        }else{
            curr->next = l2;
            l2 = l2->next;
        }
        curr = curr->next;
    }
    
    curr->next = l1 ? l1:l2;
    
    while(curr->next){
        curr = curr->next;
    }
    return curr;
}
```



#### 154. 寻找旋转排序数组中的最小值 II (二分)

```c++
int findMin(vector<int>& nums){
    int l = 0, r = nums.size() -1;
    int res = INT_MAX;
    while(l <= r){
        int mid = (l + r) >> 1;
        res = min(res, nums[mid]);
        if(nums[mid] > nums[r])
            l = mid + 1;
        else if(nums[mid] < nums[l])
            r = mid - 1;
        else
            --r; //都不满足 左移右边界
    }
    return res;
}
```



#### 435 无重叠区间（贪心）

```c++
int eraseOverlapIntervals(vector<vector<int>>& intervals) {
        sort(intervals.begin(), intervals.end(), [](vector<int>& a, vector<int>& b){
            return a[1] < b[1];
        });
        int prev = intervals[0][1];
        int cnt = 0;
        for(int i = 1; i < intervals.size(); ++i){
            if(intervals[i][0] < prev)
                ++cnt;
            else
                prev = intervals[i][1];
        }
        return cnt;
    }
```



#### 452用最少数量的箭引爆气球（与435 相似）

```c++
//匿名函数不可行，时间超过限制
static bool cmp(vector<int>& a,vector<int>& b){
        return a[1]<b[1];
}
    int findMinArrowShots(vector<vector<int>>& points) {
        int n = points.size();
        if(n ==0)
                return 0;
            sort(points.begin(),points.end(),cmp);
            int prev =points[0][1];
            int count=1;
            for(int i=1;i<n;i++){
                if(prev <points[i][0]){
                    count++;
                    prev=points[i][1];
                }
            }
            return count;
    }
```

#### 455 分发饼干（贪心）

```c++
int findContentChildren(evctor<int>& children, vector<int>& cookies){
    sort(children.begin(), children.end());
    sort(cokkies.begin(), cookies.end());
    int child = 0, int cookie = 0;
    while(child < children.size() && cookie < cookies.size()){
        if(children[child] <= cookies[cookie]) ++child;
        ++cookie;
    }
    return child;
}
```

#### 

#### 524通过删除字母匹配到字典里最长单词（归并两个有序数组的变形）

```c++
 string findLongestWord(string s, vector<string>& dictionary) {
        sort(dictionary.begin(), dictionary.end(), [](string &a, string & b){
            if(a.size() == b.size())
                return a < b;
            return a.size() > b.size();
        });
        int n = s.size();
        for(auto &ss : dictionary){
            int m = ss.size();
            int i = 0, j = 0;
            while(i < n && j < m){
                if(s[i] == ss[j])
                    ++j;
                ++i;
            }
            if(j == m)
                return ss;
        }
        return "";

    }
```

#### 540. 有序数组中的单一元素(二分)

```c++
//由于给定数组有序 且 常规元素总是两两出现，因此如果不考虑“特殊”的单一元素的话，我们有结论：成对元素中的第一个所对应的下标必然是偶数，成对元素中的第二个所对应的下标必然是奇数。


int singleNonDuplicate(vector<int>& nums) {
	int l = 0, r = nums.size() -1;
	int res;
    while(l <= r){
        int mid = (l + r) >> 1;
        if(mid & 0x1){
            if(mid + 1 < nums.size() &&nums[mid] == nums[mid + 1])
                r = mid - 1;
            else
                l = mid + 1;
        }else{
            if( mid - 1 >= 0 && nums[mid] == nums[mid - 1] && mid - 1 >= 0)
                r = mid - 1;
            else
                l = mid + 1;
        }
	}
	return nums[r];
}
```

#### 633   平方数之和（两数之和的变形）

```c++
    bool judgeSquareSum(int c) {
        long i = 0, j = sqrt(c);
        long sum;
        while(i <= j){
            sum = i * i + j * j;
            if(sum == c)
                return true;
            else if(sum > c)
                --j;
            else
                ++i; 
        }
        return false;
    }
```

#### 680验证回文串II（两数之和的变形）

```c++
 bool checkPalindrome(string &s, int low, int high){
        while(low < high){
            if(s[low] != s[high])
                return false;
            else
                ++low;
                --high;
                
        }
        return true;
    }
    bool validPalindrome(string s) {
        int low = 0, high = s.size() -1;
        while(low < high){
            if(s[low] != s[high])
                return checkPalindrome(s, low, high-1) || checkPalindrome(s,low+1, high);
            else
                ++low,--high;
        }
        return true;
    }
```

#### 763 划分字母区间 (贪心)

```c++
vector<int> partitionLabels(string s) {
        int last[26];
        int i = 0, n = s.size();
        for(;i < n; i++)
            last[s[i] - 'a'] = i;
        int start = 0, end = 0;
        vector<int>res;
        for(i = 0; i < n; ++i)
        {
            end = max(end, last[s[i] - 'a']);
            if(i == end)
            {
                res.push_back(end -start + 1);
                start = end + 1;
            }
        }
        return res;
    }
```

### 常用排序

#### 快速排序

```c++
void quick_sort(vector<int>& nums, int l, int r){
    if(l + 1 >= r)
        return;
    int first = l, last = r -1, key = nums[first];
    while(first < last){
        while(first < last && nums[last] >= key)
            --last;
  
        if(first < last)
    		nums[first] = nums[last];
    	while(first < last && nums[first] <= key)
        	++first;
        if(first < last)
    		nums[last] = nums[first];
    }
	nums[first] = key;
	quick_sort(nums, l, first - 1);
	quick_sort(nums, first + 1, l);
}
```

#### 归并排序

```c++
void merge_sort(vector<int> &nums, int l, int r, vector<int> &temp){
    if(l + 1 >= r)
        return;
    
    int m = l + (r -l)/2;
    merge_sort(nums, l, m, temp);
    mergr_sort(nums, m, r, temp);
    
    int p = l, q = m, i = l;
    while(p < m || q < r){
        if(q >= r || ((p < m) && nums[p] <= nums[q])){
            temp[i++] = nums[p++];
        }else{
            temp[i++] = nums[q++];
        }
    }
    for(i = l; i < r; ++i)
        nums[i] = temp[i]
}
    
```

#### 插入排序

```c++
void insertion_sort(vector<int>&nums, int n){
    for(int i = 0; i < n; ++i){
        for(int j = i; j > 0 && nums[j] < nums[j -1]; --j){
            swap(nums[j], nums[j-1]);
        }
    }
}
```

#### 冒泡排序

```c++
void bubble_sort(vector<int>&nums, int n){
    bool swapped;
    for(int i = 1; i < n 1; ++i){
        swapped = false;
        for(int j = i; j < n - i - 1; ++j){
            if(nums[j] < nums[j -1]){
                swap(nums[j], nums[j -1]);
                swapped = true;
            }
        }
        if(!swapped)
            break;
    }
}
```

#### 选择排序

```c++
void selection_sort(vector<int>&nums, int n){
    int mid;
    for(int i = 0; i < n - 1; ++i){
        mid = i;
        for(int j = i+1; j < n; ++j){
            if(nums[j] < nums[mid]){
                mid = j;
            }
        }
        swap(nums[mid], nums[i]);
    }
}
    
```

```c++
//以上排序的调用
void sort(){
    vector<int> nums = {1, 2, 3, 6, 4, 9, 7};
    vector<int> temp(nums.size());
    sort(nums.begin(), nums.end());
    quick_sort(nums, 0, nums.size());
    merge_sort(nums, 0, nums.size());
    insertin_sort(nums, nums.size());
    bubble_sort(nums, nums.size());
    selection_sort(nums, nums.size());
}
```

```c
while(len > 0){
  va0 = PGROUNDDOWN(dstva);
  pa0 = walkaddr(pagetable, va0);

  // 处理COW页面的情况
  if(cowpage(pagetable, va0) == 0) {
    // 更换目标物理地址
    pa0 = (uint64)cowalloc(pagetable, va0);
  }

  if(pa0 == 0)
    return -1;

  ...
}
```
