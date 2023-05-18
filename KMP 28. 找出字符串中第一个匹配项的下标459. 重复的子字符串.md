## 28. 找出字符串中第一个匹配项的下标

```c++
给你两个字符串 haystack 和 needle ，请你在 haystack 字符串中找出 needle 字符串的第一个匹配项的下标（下标从 0 开始）。如果 needle 不是 haystack 的一部分，则返回  -1 。

示例 1：

输入：haystack = "sadbutsad", needle = "sad"
输出：0
解释："sad" 在下标 0 和 6 处匹配。
第一个匹配项的下标是 0 ，所以返回 0 。
示例 2：

输入：haystack = "leetcode", needle = "leeto"
输出：-1
解释："leeto" 没有在 "leetcode" 中出现，所以返回 -1 。
```

### KMP

#### KMP的主要思想是当出现字符串不匹配时，可以知道一部分之前已经匹配的文本内容，可以利用这些信息避免从头再去做匹配了。

### 前缀表

前缀表是用来**回退**的，它记录了 **当模式串与主串(文本串)不匹配**的时候，模式串应该**从哪里开始重新匹配**。

前缀表：记录下标i之前（包括i）的字符串中，有**多大长度的相同前缀后缀**。**相同前后缀的长度**

​	前缀 是指**不包含最后一个字符**的**所有以第一个字符开头的连续子串。**

​	后缀 是指**不包含第一个字符**的**所有以最后一个字符结尾的连续子串。**

### 如何计算前缀表

模式串与前缀表对应位置的数字表示的就是：下标i之前（包括i）的字符串中，有多大长度的相同前缀后缀。

#### 构造以前缀表统一减一之后的next数组

```c++
void getNext(int* next, const string& s){
    //定义两个指针i和j，j指向前缀末尾位置，i指向后缀末尾位置
    //对next数组进行初始化赋值，如下：选择j初始化为-1 前缀表统一减一
    //next[i] 表示 i（包括i）之前最长相等的前后缀长度（其实就是j）
    int j = -1;
    next[0] = j;
    //因为j初始化为-1，那么i就从1开始，进行s[i] 与 s[j+1]的比较。    即 从 第二个字符s[i] 和 第一个字符s[j+1] 开始比较
    //所以遍历模式串s的循环下标i 要从 1开始
    for(int i = 1; i < s.size(); i++) { // 注意i从1开始
        while (j >= 0 && s[i] != s[j + 1]) {
            // 如果 s[i] 与 s[j+1]不相同，也就是遇到 前后缀末尾不相同的情况，就要向前回退。
            
            //next[j]就是记录着j（包括j）之前的子串的相同前后缀的长度。

			//那么 s[i] 与 s[j+1] 不相同，就要找 j+1前一个元素 j 在next数组里的值（就是next[j]）
            
            j = next[j]; // 向前回退\
            
        }
        if (s[i] == s[j + 1]) { // 如果 s[i] 与 s[j + 1] 相同，那么就同时向后移动i 和j   说明找到了相同的前后缀
            j++;
        }
        next[i] = j; // 将j（前缀的长度）赋给next[i]
    }
}


void getNext(const string &s, vector<int> &next)
    {
        //初始化
        // j+1 是 i 之前(包括i)的字符串的最长公共前后缀长度 s[j+1]即为最长公共前后缀的前缀末尾
        int j = -1; 
        next[0] = j;

        for(int i = 1; i < s.size(); ++i)   //遍历模式串s 为每个字符计算next数组的值
        {//当前字符s[i](假设是公共后缀末尾)和s[j+1](假设公共前缀末尾)不匹配  j往前回退 
            while(j+1 > 0 && s[i] != s[j+1]) 
                j = next[j+1 - 1];
            if(s[i] == s[j+1]) //r如果是因为匹配上了的原因退出循环 最长公共前后缀长度++
                j++;
            next[i] = j;
        }
    }
    
   
```

### 使用next数组来做匹配

在文本串s里 找是否出现过模式串t。

定义两个下标j 指向模式串起始位置，i指向文本串起始位置。

那么j初始值依然为-1，**因为next数组里记录的起始位置为-1。**

i就从0开始，遍历文本串，代码如下：

```cpp
for (int i = 0; i < s.size(); i++) 
```

接下来就是 s[i] 与 t[j + 1] （因为j从-1开始的） 进行比较。

如果 s[i] 与 t[j + 1] 不相同，j就要从next数组里寻找下一个匹配的位置。

代码如下：

```cpp
while(j >= 0 && s[i] != t[j + 1]) {
    j = next[j];
}
```

如果 s[i] 与 t[j + 1] 相同，那么i 和 j 同时向后移动， 代码如下：

```cpp
if (s[i] == t[j + 1]) {
    j++; // i的增加在for循环里
}
```

如何判断在文本串s里出现了模式串t呢，如果j指向了模式串t的末尾，那么就说明模式串t完全匹配文本串s里的某个子串了。

本题要在文本串字符串中找出模式串出现的第一个位置 (从0开始)，所以返回当前在文本串匹配模式串的位置i 减去 模式串的长度，就是文本串字符串中出现模式串的第一个位置。

代码如下：

```cpp
if (j == (t.size() - 1) ) {
    return (i - t.size() + 1);
}
```

### 前缀表统一减一 C++代码实现

```cpp

class Solution {
public: 
    void getNext(int *next, const string s){//构造next前缀表
        int j = -1;//初始化j  前缀末尾
        next[0] = j;
        for(int i = 1; i < s.size(); i++){ //i 后缀末尾 从第二个字符开始遍历
            // s[i] 和 s[j+1] 不相同  j回退 
            while(j>=0 && s[i] != s[j+1])
                j = next[j];//当前比较的字符是s[j+1]  找到前一位的next即为j
            //如果相同 j++  即最长相等前后缀长度加一
            if(s[i] == s[j+1])
                j++;
            
            next[i] = j; //此时i的next数组的值即为j的值
        }

    }
    int strStr(string haystack, string needle) {
        //先给needle数组构建前缀表
        int size = needle.size();
        int Next[size]; //创建大小给定的数组
        getNext(Next, needle);
        int k = -1;
        for(int i = 0; i < haystack.size(); i++){//从haystack的第一个元素开始比较
            while( k >= 0 && haystack[i] != needle[k+1])
                k = Next[k];
            if(haystack[i] == needle[k+1])
                k++;
            if( k == size - 1) //// 文本串haystack里出现了模式串needle
                return i - k;
        }
        return -1;

    }
};
```





------

## 459. 重复的子字符串

```cpp
给定一个非空的字符串 s ，检查是否可以通过由它的一个子串重复多次构成。
    
示例 1:
输入: s = "abab"
输出: true
解释: 可由子串 "ab" 重复两次构成。
    
示例 2:
输入: s = "aba"
输出: false
    
示例 3:
输入: s = "abcabcabcabc"
输出: true
解释: 可由子串 "abc" 重复四次构成。 (或子串 "abcabc" 重复两次构成。)
```

**数组长度减去最长相同前后缀的长度相当于是第一个周期的长度，也就是一个周期的长度，如果这个周期可以被整除，就说明整个数组就是这个周期的循环。**

##### 当一个字符串由重复子串组成的，最长相等前后缀不包含的子串就是最小重复子串。

##### 整个字符串的最长相等前后缀长度为 (next[len-1]+1) 这里的next数组是以统一减一的方式计算的，因此需要+1

##### 数组长度减去最长相同前后缀的长度相当于是第一个周期的长度，(len - (next[len-1]+1))也就是一个周期的长度

#### 如果这个周期可以被整除，就说明整个数组就是这个周期的循环。

```cpp
class Solution {
public:
    void getNext(int *next, const string s){//构造next前缀表
        int j = -1;//初始化j  前缀末尾
        next[0] = j;
        for(int i = 1; i < s.size(); i++){ //i 后缀末尾 从第二个字符开始遍历
            // s[i] 和 s[j+1] 不相同  j回退 
            while(j>=0 && s[i] != s[j+1])
                j = next[j];//当前比较的字符是s[j+1]  找到前一位的next即为j
            //如果相同 j++  即最长相等前后缀长度加一
            if(s[i] == s[j+1])
                j++;
            
            next[i] = j; //此时i的next数组的值即为j的值
        }
    }
    //当一个字符串由重复子串组成的，最长相等前后缀不包含的子串就是最小重复子串。
    //整个字符串的最长相等前后缀长度为 (next[len-1]+1) 
    //这里的next数组是以统一减一的方式计算的，因此需要+1
    //数组长度减去最长相同前后缀的长度相当于是第一个周期的长度，(len - (next[len-1]+1))
    //也就是一个周期的长度，如果这个周期可以被整除，就说明整个数组就是这个周期的循环。
    bool repeatedSubstringPattern(string s) { 
        if(!s.size())
            return false;
        int next[s.size()];
        getNext(next, s);
        int len = s.size();
        if(next[len-1]!=-1 && len%(len - (next[len-1]+1)) == 0)
            return true;
        
        return false;

    }
    
    
    bool repeatedSubstringPattern(string s) {
        string str = s + s;         //拼接两个s
        vector<int> next(str.size());//构造str的next
        getNext(str, next);
        int max = next[str.size() - 1] + 1; //str的最长公共前后缀长度
        //最长相等前后缀不包含的子串就是最小重复子串。
        int len = str.size() - max;         //str中不在最长公共前后缀中的子串
        //如果可以由一个子串重复多次构成 就可以整除这个字串的长度
        if(s.size()%len==0 && s.size()/len > 1) 
            return true;
        else
            return false;


    }
};

};
```



