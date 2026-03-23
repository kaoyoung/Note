## Q1 : 以下程式如何更快
```C
int* count_satisfying_node(struct TreeNode* root, int *satisfy_node_number){
    if(root == NULL){
        int* result = malloc(2*sizeof(int));
        result[0] = 0;
        result[1] = 0;
        return result;
    }  

    int* left_subtree = count_satisfying_node(root->left, satisfy_node_number);
    int* right_subtree = count_satisfying_node(root->right, satisfy_node_number);
    int current_sum = left_subtree[0] + right_subtree[0] + root->val;
    int current_node = left_subtree[1] + right_subtree[1] + 1;
    *satisfy_node_number += ((current_sum/current_node) == root->val);
    free(left_subtree);
    free(right_subtree);
  
    int* result = malloc(2*sizeof(int));
    result[0] = current_sum;
    result[1] = current_node;
    return result;
}

int averageOfSubtree(struct TreeNode* root) {
    int satisfy_node_number = 0;
    count_satisfying_node(root, &satisfy_node_number);
    return satisfy_node_number;

}
```
## A1 :
```C
typedef struct node_result{
    int current_value;
    int current_node;
}node_result;

node_result count_satisfying_node(struct TreeNode* root, int *satisfy_node_number){
    if(root == NULL){
        node_result result = {0, 0};
        return result;
    }

    node_result left_subtree = count_satisfying_node(root->left, satisfy_node_number);
    node_result right_subtree = count_satisfying_node(root->right, satisfy_node_number);

    int current_sum = left_subtree.current_value + right_subtree.current_value + root->val;
    int current_node = left_subtree.current_node + right_subtree.current_node + 1;

    *satisfy_node_number += ((current_sum/current_node) == root->val);
    node_result result = {current_sum, current_node};
    return result;
}
  
int averageOfSubtree(struct TreeNode* root) {
    int satisfy_node_number = 0;
    count_satisfying_node(root, &satisfy_node_number);
    return satisfy_node_number;
}
```
**關鍵想法 :** stack跟heap都是記憶體配置，但速度有差距。
### 細說stack跟heap的記憶體配置差距
#### 分配機制 : 
- stack : 移動stack pointer。
- heap : 系統必須遍歷一個「可用空間清單 (Free List)」，尋找一塊「夠大且合適」的空地。如果找不到，還要跟作業系統要更多記憶體，甚至需要重組碎片。這涉及 **成百上千個 CPU 指令**。
#### CPU 快取 (Cache Locality)
- stack : 資料緊湊 (Contiguous) Stack 上的變數是連續存放的。較高的 Cache Hit Rate。
- heap  : 資料破碎 (Fragmented) `malloc` 出來的記憶體可能散落在 RAM 的各個角落。
#### 多執行緒安全 (Thread Safety)
- stack : 每個執行緒 (Thread) 都有自己獨立的 Stack。
- heap : 是所有執行緒共用的。
### 在c中`free()`可以針對在stack的變數嗎 ?
>在 C 語言中，`free()` 函數**只能**用來釋放由 `malloc()`、`calloc()` 或 `realloc()` 所分配的堆積(Heap)記憶體。如果你嘗試對堆疊(Stack)上的變數（即一般的區域變數）使用 `free()`，會導致程式崩潰或產生未定義行為（Undefined Behavior）。

---
## Q2 : 以下程式如何有何問題
```c
void build_mergeTree(const struct TreeNode *const root1, const struct TreeNode *const root2, struct TreeNode* new_node){
    if(root1 == NULL && root2 == NULL){
        new_node = NULL;    
        free(new_node);            
        return;
    }
    else if(root1 != NULL && root2 == NULL){
        new_node->val = root1->val;
    }
    else if(root1 == NULL && root2 != NULL){
        new_node->val = root2->val;
    }
    else{
        new_node->val = root1->val + root2->val;
    } 

    struct TreeNode *new_left_node = calloc(1, sizeof(struct TreeNode));
    struct TreeNode *new_right_node = calloc(1, sizeof(struct TreeNode));
    new_node->left = new_left_node;
    new_node->right = new_right_node;
    if(root1 == NULL){
        build_mergeTree(NULL, root2->left, new_node->left);
        build_mergeTree(NULL, root2->right, new_node->right);
    }
    else if(root2 == NULL){
        build_mergeTree(root1->left, NULL, new_node->left);
        build_mergeTree(root1->right, NULL, new_node->right);
    }
    else{
        build_mergeTree(root1->left, root2->left, new_node->left);
        build_mergeTree(root1->right, root2->right, new_node->right);
    }    
    return;
}

struct TreeNode* mergeTrees(struct TreeNode* root1, struct TreeNode* root2) {
    struct TreeNode *new_root = malloc(sizeof(struct TreeNode));
    build_mergeTree(root1, root2, new_root);
    return new_root;
}
```
題目為[617. Merge Two Binary Trees](https://leetcode.com/problems/merge-two-binary-trees/)
## A2 : 
- 要跟改變數傳指針，要跟改指針傳指針的指針。
- 記憶體先`free()`在指定為NULL。
```c
void build_mergeTree(const struct TreeNode *const root1, const struct TreeNode *const root2, struct TreeNode **new_node){
    if(root1 == NULL && root2 == NULL){
        *new_node = NULL;    
        free(*new_node);
        return;
    }
    else if(root1 != NULL && root2 == NULL){
        (*new_node)->val = root1->val;
    }
    else if(root1 == NULL && root2 != NULL){
        (*new_node)->val = root2->val;
    }
    else{
        (*new_node)->val = root1->val + root2->val;
    }
  
    struct TreeNode *new_left_node = calloc(1, sizeof(struct TreeNode));
    struct TreeNode *new_right_node = calloc(1, sizeof(struct TreeNode));
    (*new_node)->left = new_left_node;
    (*new_node)->right = new_right_node;
    if(root1 == NULL){
        build_mergeTree(NULL, root2->left, &((*new_node)->left));
        build_mergeTree(NULL, root2->right, &((*new_node)->right));
    }
    else if(root2 == NULL){
        build_mergeTree(root1->left, NULL, &((*new_node)->left));
        build_mergeTree(root1->right, NULL, &((*new_node)->right));
    }
    else{
        build_mergeTree(root1->left, root2->left, &((*new_node)->left));
        build_mergeTree(root1->right, root2->right, &((*new_node)->right));
    }    
    return;
}

  

struct TreeNode* mergeTrees(struct TreeNode* root1, struct TreeNode* root2) {
    struct TreeNode *new_root = calloc(1, sizeof(struct TreeNode));
    build_mergeTree(root1, root2, &new_root);
    return new_root;
}
```
這寫法可以優化，變成先確認可以分配再分配，而不是分配後再取消。
```c
struct TreeNode* mergeTrees(struct TreeNode* root1, struct TreeNode* root2) {
    if(root1 == NULL && root2 == NULL){
        return NULL;
    }  

    struct TreeNode *node = calloc(1, sizeof(struct TreeNode));
    if(root1 != NULL && root2 == NULL){
        node->val = root1->val;
        node->left = mergeTrees(root1->left, NULL);
        node->right = mergeTrees(root1->right, NULL);
    }
    else if(root1 == NULL && root2 != NULL){
        node->val = root2->val;
        node->left = mergeTrees(NULL, root2->left);
        node->right = mergeTrees(NULL, root2->right);
    }
    else{
        node->val = root1->val + root2->val;
        node->left = mergeTrees(root1->left, root2->left);
        node->right = mergeTrees(root1->right, root2->right);
    }  

    return node;
}
```

---
## Q3 : 讓以下程式更簡潔
```c
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     struct TreeNode *left;
 *     struct TreeNode *right;
 * };
 */
int get_str_length(struct TreeNode* root){
    if(root == NULL){
        return 0;
    }
  
    int add_number = 0;
    if(root->left != NULL){
        add_number += 2;
    }
    if(root->left == NULL &&  root->right != NULL){
        add_number += 4;
    }
    else if(root->left != NULL &&  root->right != NULL){
        add_number += 2;
    }
  
    add_number += (root->val < 0);
    int abs_val = abs(root->val);
    if(0<= abs_val && abs_val<=9){
        add_number += 1;
    }
    else if(10<= abs_val && abs_val<=99){
        add_number += 2;
    }
    else if(100<= abs_val && abs_val<=999){
        add_number += 3;
    }
    else if(abs_val == 1000){
        add_number += 4;
    }
    return  add_number + get_str_length(root->left) + get_str_length(root->right);
}

  

void get_parentheses_str(const struct TreeNode *const root, char *cur_string, int *cur_offset){
    if(root == NULL){
        return;
    }

    if(root->val < 0){
        cur_string[(*cur_offset)++] = '-';
    }
    int root_value = abs(root->val);
    int string_length = 0;
    if(root_value == 0){
        cur_string[(*cur_offset)++] = '0';
        ++string_length;
    }
    while(root_value > 0){
        cur_string[(*cur_offset)++] = '0'+(root_value%10);
        root_value /= 10;
        ++string_length;
    }
  
    for(int i=0; i<string_length/2; ++i){
        char temp_char = cur_string[*cur_offset-1-i];
        cur_string[*cur_offset-1-i] = cur_string[*cur_offset-string_length+i];
        cur_string[*cur_offset-string_length+i] = temp_char;
    }  
  
    if(root->left != NULL){    
        cur_string[(*cur_offset)++] = '(';
        get_parentheses_str(root->left, cur_string, cur_offset);
        cur_string[(*cur_offset)++] = ')';
    }
    if(root->right != NULL){        
        if(root->left == NULL){
            cur_string[(*cur_offset)++] = '(';
            cur_string[(*cur_offset)++] = ')';
        }
        cur_string[(*cur_offset)++] = '(';
        get_parentheses_str(root->right, cur_string, cur_offset);
        cur_string[(*cur_offset)++] = ')';
    }
    return;
}  

char* tree2str(struct TreeNode* root) {
    int str_length = get_str_length(root);
    char *return_str = calloc(str_length+1, sizeof(char));
    int cur_offset = 0;
    get_parentheses_str(root, return_str, &cur_offset);
    return return_str;
}
```
題目[606. Construct String from Binary Tree](https://leetcode.com/problems/construct-string-from-binary-tree/description/?envType=problem-list-v2&envId=v4rmerev)
## A3 : 
這程式有重複邏輯
- `get_str_length()` 跟`get_parentheses_str` 推offset的流程重複了
處理太多底層的東西
- 善用`snprintf()`和`sprintf()`這兩函式 
	- `int sprintf(char *str, const char *format, ...)` 
		- `str` : 这是指向一个字符数组的指针，该数组存储了 C 字符串。
		- `format` : 这是字符串，包含了要被写入到字符串 str 的文本。
	- `int snprintf(char *buffer, size_t n, const char *format, ...);`
		- `buffer`：接收格式化字符串的字符数组（缓冲区）。
		- `n`：指定 `buffer` 的总大小（包括用于终止的 `\0`）。
		- `format`：格式化字符串（例如 `"Hello, %s! Your age is %d."`）。
	- 兩函數返回值 : 
		- 如果成功，则返回写入的字符总数，不包括字符串追加在字符串末尾的空字符。
		- 如果失败，则返回一个负数。
例子 : 
```c
#include <stdio.h>  
#include <math.h>  
  
int main()  
{  
   char str[80];  
   sprintf(str, "Pi 的值 = %f", M_PI);  
   
   return(0);  
}
```
### **修改原問題的程式**
```c
void get_parentheses_str(const struct TreeNode *const root, char *cur_string, int *cur_offset){
    if(root == NULL){
        return;
    }  

    if(cur_string == NULL){
        *cur_offset += snprintf(cur_string, 0, "%d", root->val);
    }
    else{
        *cur_offset += sprintf(cur_string+*cur_offset, "%d", root->val);
    }

    if(root->left != NULL || root->right != NULL){    
        if(cur_string != NULL) cur_string[*cur_offset] = '(';
        ++(*cur_offset);
        get_parentheses_str(root->left, cur_string, cur_offset);
        if(cur_string != NULL) cur_string[*cur_offset] = ')';
        ++(*cur_offset);
    }
    if(root->right != NULL){       
        if(cur_string != NULL) cur_string[*cur_offset] = '(';
        ++(*cur_offset);
        get_parentheses_str(root->right, cur_string, cur_offset);
        if(cur_string != NULL) cur_string[*cur_offset] = ')';
        ++(*cur_offset);
    }

    return;
}

  

char* tree2str(struct TreeNode* root) {
    int str_length = 0;
    get_parentheses_str(root, NULL, &str_length);
    char *return_str = calloc(str_length+1, sizeof(char));

    int cur_offset = 0;
    get_parentheses_str(root, return_str, &cur_offset);
    return return_str;
}
```
--- 
## Q4 : 如何用c庫函數的`qsort()`
## A4 : 
```c
void qsort(void *base, size_t nitems, size_t size, int (*compar)(const void *, const void *));
```
- base : 指向待排序数组的第一个元素的指针。
- nitems : 数组中的元素数量。
- size : 数组中每个元素的大小（以字节为单位）。
- compare : 比较函数应当返回一个整数，表示比较结果：
	- 小于零：表示第一个元素小于第二个元素。
	- 等于零：表示两个元素相等。
	- 大于零：表示第一个元素大于第二个元素。
---
## Q5 : 讓以下程式更快
```C
int comparefnt(const void *a, const void *b){
    int temp_a = *(int *)a;
    int temp_b = *(int *)b;
    return temp_a - temp_b;
}

int* collect_subtree_sum(struct TreeNode* root, int* returnSize, int *returnSum){
    if(root == NULL){
        *returnSize = 0;
        return NULL;
    }

    if(root->left == NULL && root->right == NULL){
        *returnSize = 1;
        *returnSum = root->val;
        int *returnArray = calloc(1, sizeof(int));
        returnArray[0] = root->val;
        return returnArray;
    }  

    int leftSize = 0, leftSum = 0;
    int *left_array = collect_subtree_sum(root->left, &leftSize, &leftSum);
    int rightSize = 0, rightSum = 0;
    int *right_array = collect_subtree_sum(root->right, &rightSize, &rightSum);
    int totalSize = leftSize + rightSize + 1;
    int *returnArray = calloc(totalSize, sizeof(int));
    for(int i=0; i<leftSize; ++i){
        returnArray[i] = left_array[i];
    }
    for(int i=0; i<rightSize; ++i){
        returnArray[leftSize+i] = right_array[i];
    }
    returnArray[totalSize-1] = root->val + leftSum + rightSum;
	free(left_array);
	free(right_array);  

    *returnSize = totalSize;
    *returnSum = root->val + leftSum + rightSum;
    return returnArray;
} 
  
int* findFrequentTreeSum(struct TreeNode* root, int* returnSize) {
    int totalSize = 0, totalSum = 0;
    int *subtreeSumArray = collect_subtree_sum(root, &totalSize, &totalSum);
  
    qsort(subtreeSumArray, totalSize, sizeof(int), comparefnt);
    if(totalSize == 0){
        return NULL;
    }    
  
    int* answer = calloc(10000, sizeof(int));
    int size = 0;
    int frequent_number = subtreeSumArray[0];
    int max_count = 1;
    int cur_count = 0;
    for(int i=0; i<totalSize; ++i){
        if(frequent_number == subtreeSumArray[i]){
            ++cur_count;
        }
        else{
            cur_count = 1;
            frequent_number = subtreeSumArray[i];
        }
        if(cur_count == max_count){
            answer[size] = subtreeSumArray[i];           
            size += 1;
        }
        else if(cur_count > max_count){
            answer[0] = subtreeSumArray[i];
            size = 1;
            max_count = cur_count;
        }
    }

    *returnSize = size;
    int *final_answer = calloc(size, sizeof(int));
    for(int i=0; i<size; ++i){
        final_answer[i] = answer[i];
    }
    free(answer);
    return final_answer;
}
```
## A5 : 
修改建議 : 
- 先分配好記憶體
- 用完的`malloc()`，記得`free()`
```c=
int max(const int a, const int b){
    return (a>b)? a:b;
}

int comparefnt(const void *a, const void *b){
    long long arg1 = *(const int *)a;
    long long arg2 = *(const int *)b;
    if (arg1 < arg2) return -1;
    if (arg1 > arg2) return 1;
    return 0;
}

int count_node_number(struct TreeNode* root){
    if(root == NULL){
        return 0;
    }

    return count_node_number(root->left) + count_node_number(root->right) + 1;
}

int get_all_subtree_sum(struct TreeNode* root, int *sum_table, int *offset){
    if(root == NULL){
        return 0;
    }

    int current_sum = get_all_subtree_sum(root->left, sum_table, offset) + get_all_subtree_sum(root->right, sum_table, offset) + root->val;
    sum_table[*offset] = current_sum;
    *offset += 1;
    return current_sum;
} 

int* findFrequentTreeSum(struct TreeNode* root, int* returnSize) {
    int tree_size = count_node_number(root);
    int *sum_table = calloc(tree_size, sizeof(int));
    int offset = 0;
    get_all_subtree_sum(root, sum_table, &offset);

    qsort(sum_table, tree_size, sizeof(int), comparefnt);

    int cur_frequence = 0;
    int most_freq_number = INT_MIN;
    int max_frequence = 0;
    for(int i=0; i<tree_size; ++i){
        if(sum_table[i] != most_freq_number){
            cur_frequence = 1;
            most_freq_number = sum_table[i];
        }
        else{
            ++cur_frequence;
        }
  
        max_frequence = max(max_frequence, cur_frequence);
    }  

    cur_frequence = 0;
    int max_freq_number = 0;
    for(int i=0; i<tree_size; ++i){
        if(sum_table[i] != most_freq_number){
            cur_frequence = 1;
            most_freq_number = sum_table[i];
        }
        else{
            ++cur_frequence;
        }  

        if(cur_frequence == max_frequence){
            ++max_freq_number;
        }
    } 

    int *answer = calloc(max_freq_number, sizeof(int));
    int index = 0;
    cur_frequence = 0;
    for(int i=0; i<tree_size; ++i){
        if(sum_table[i] != most_freq_number){
            cur_frequence = 1;
            most_freq_number = sum_table[i];
        }
        else{
            ++cur_frequence;
        }  

        if(cur_frequence == max_frequence){
            answer[index++] = sum_table[i];
        }
    }
  
    free(sum_table);
    *returnSize = max_freq_number;
    return answer;
}
```
---
## Q6 : Show that an in-order traversal of a BST produces an array in ascending order ?
## A6 :
回顧BST的性質，對於一個節點來說，左孩子的值小於自己的值，自己的值小於右孩子的值，所以最左的值最小。
我想用數學歸納法證。對於第一個點，他是最左邊的點所以最小。現在只需要證每一個點(不包含第一個點)都比前一個點大就完事。依照in-order traversal，對於一個點來說，前一個點可以來自左子樹，或是父節點而你是這父節點的右子樹，所以依照BST的定義，符合每一個點(不包含第一個點)都比前一個點大。
### 這說明的瑕疵
#### 1. 歸納對象不明確
數學歸納法（Mathematical Induction）通常需要一個明確的對象。在樹狀結構中，我們通常針對 **「樹的節點數 $n$」** 或 **「樹的高度 $h$」** 來做歸納。你的說法比較像是在遍歷的序列（Sequence）上做歸納，這會讓處理「父節點與子樹關係」時變得非常複雜。
#### 2. 基底情況（Base Case）不夠完整
你提到「第一個點是最左邊所以最小」，但在歸納法中，基底情況通常是 $n=1$（只有一個根節點的樹）或 $n=0$（空樹）。
#### 3. 遞迴性質未充分利用
BST 的定義本身就是遞迴的。證明「In-order 產生遞增序列」最優雅的方式是利用 **結構歸納法（Structural Induction）**。
### 建議的標準證明結構
若要完美回答這題，建議採用以下邏輯。
**假設 $P(n)$ 是指：對於一個擁有 $n$ 個節點的 BST，其 In-order 遍歷結果為遞增序列。**
#### 1. 基底情況 (Base Case)
當 $n = 1$ 時，樹中只有一個節點。In-order 遍歷序列只包含該節點本身，單一元素的序列顯然是遞增的。$P(1)$ 成立。
#### 2. 歸納假設 (Inductive Hypothesis)
假設對於所有節點數少於 $k$ 的 BST，$P(n)$ 皆成立（即其 In-order 遍歷均為遞增序列）。
#### 3. 歸納步驟 (Inductive Step)
考慮一個擁有 $k$ 個節點的 BST，設其根節點為 $R$，左子樹為 $L$，右子樹為 $R_{tree}$。
- **根據 In-order 定義：** 遍歷序列為 `InOrder(L) + [R] + InOrder(R_tree)`。    
- **根據歸納假設：** `InOrder(L)` 和 `InOrder(R_tree)` 各自都是遞增序列。    
- 根據 BST 性質： 1. 左子樹的所有節點值都小於 $R$ $\Rightarrow$ InOrder(L) 的最後一個元素（最大值）小於 $R$。2. 右子樹的所有節點值都大於 $R$ $\Rightarrow$ $R$ 小於 InOrder(R_tree) 的第一個元素（最小值）。
- **結論：** 因為 `左序列(遞增) < R < 右序列(遞增)`，組合起來的完整序列必然也是遞增的。
### 證明差別
原始證明侷限在單一點的操作上，這使得證明過度依賴於in-order traversal的操作細節上。改進後的證明抽離了in-order traversal的操作細節，專注於整個樹的架構上，巧妙的用root分離出小於k的子樹，然後直接套數歸的假設。
>回歸樹的基本想法 :  左子樹 + root + 右子樹
### 用途
- 在BST中找任兩點最小的距離。
- 在BST中找某個數值區間的點集
---
## Q7 : Where should you insert the new node



---
## Q8 : Where should you delete the node



---
## Q9 : How to convert the recursive version into iteraion version and vise versa
First take a look at the following problem 
- https://leetcode.com/problem-list/v44k6xlv/
### 先想前/中/後序的情況
Stack 中存入節點時，多存一個「標記 (Flag)」或「顏色」：
- **0 (White / Visit)**: 代表第一次看到它，還沒處理它的子節點（還不能打印）。    
- **1 (Gray / Process)**: 代表子節點都處理完了（或者不需要處理子節點），現在輪到處理它自己（打印/存入結果）。
1. 前序 (Preorder): 中 -> 左 -> 右
```c++
vector<int> preorderTraversal(TreeNode* root) {
    vector<int> res;
    if (!root) return res;
    
    stack<pair<TreeNode*, int>> st;
    st.push({root, 0}); // 0 代表尚未處理

    while (!st.empty()) {
        auto [node, type] = st.top();
        st.pop();

        if (node == nullptr) continue;

        if (type == 1) {
            res.push_back(node->val); // Process
        } else {
            // Preorder: 中 -> 左 -> 右
            // Stack 入序: 右 -> 左 -> 中 (因為 Stack 是後進先出)
            st.push({node->right, 0}); // 右
            st.push({node->left, 0});  // 左
            st.push({node, 1});        // 中 (標記為 1，下次遇到就打印)
        }
    }
    return res;
}
```
2. 中序 (Inorder): 左 -> 中 -> 右
```c++
// ... (同上) ...
        } else {
            // Inorder: 左 -> 中 -> 右
            // Stack 入序: 右 -> 中 -> 左
            st.push({node->right, 0}); // 右
            st.push({node, 1});        // 中 (準備打印)
            st.push({node->left, 0});  // 左
        }
        // ... (同上) ...
```
3. 後序 (Postorder): 左 -> 右 -> 中
```c++
// ... (同上) ...
        } else {
            // Postorder: 左 -> 右 -> 中
            // Stack 入序: 中 -> 右 -> 左
            st.push({node, 1});        // 中 (最後打印)
            st.push({node->right, 0}); // 右
            st.push({node->left, 0});  // 左
        }
        // ... (同上) ...
```
#### 經典派模板 (Classic Optimized Approach)
1. 經典前序 (Preorder)
```c++
vector<int> preorderTraversal(TreeNode* root) {
    vector<int> res;
    if (!root) return res;
    stack<TreeNode*> st;
    st.push(root);
    while (!st.empty()) {
        TreeNode* node = st.top(); st.pop();
        res.push_back(node->val);
        if (node->right) st.push(node->right); // 先壓右
        if (node->left) st.push(node->left);   // 後壓左 -> 左先出
    }
    return res;
}
```
2. 經典中序 (Inorder)
```c++
vector<int> inorderTraversal(TreeNode* root) {
    vector<int> res;
    stack<TreeNode*> st;
    TreeNode* cur = root;
    
    while (cur != nullptr || !st.empty()) {
        // 1. 一路向左鑽到底，沿途入棧
        while (cur != nullptr) {
            st.push(cur);
            cur = cur->left;
        }
        // 2. 左邊沒路了，彈出一個節點並處理
        cur = st.top(); st.pop();
        res.push_back(cur->val);
        
        // 3. 轉向右子樹
        cur = cur->right;
    }
    return res;
}
```
3. 經典後序 (Postorder)
```c++
vector<int> postorderTraversal(TreeNode* root) {
    vector<int> res;
    stack<TreeNode*> st;
    TreeNode* cur = root;
    TreeNode* prev = nullptr; // 記錄上一個「已經被打印」的節點

    while (cur != nullptr || !st.empty()) {
        // 1. 一路向左
        while (cur != nullptr) {
            st.push(cur);
            cur = cur->left;
        }
        
        // 2. 查看 Stack 頂端 (還不能 pop，因為可能還有右子樹沒走)
        TreeNode* topNode = st.top();
        
        // 3. 判斷是否能處理當前節點：
        //    a. 沒有右子樹 (topNode->right == nullptr)
        //    b. 或者右子樹剛剛處理過了 (topNode->right == prev)
        if (topNode->right == nullptr || topNode->right == prev) {
            res.push_back(topNode->val);
            st.pop();      // 真的處理完了，可以 pop
            prev = topNode; // 更新 prev
            cur = nullptr; // 重要！設為 nullptr 避免下個迴圈又重複往左鑽
        } else {
            // 4. 還有右子樹沒走，轉向右邊
            cur = topNode->right;
        }
    }
    return res;
}
```

