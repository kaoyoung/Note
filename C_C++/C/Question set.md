## Q1:  What if you write `*offset++` instead of `++*offset` in line 19
```c= ln:true
int find_tree_height(const struct TreeNode* root){
    if(root == NULL){
        return 0;
    }  

    return 1 + max(find_tree_height(root->left), find_tree_height(root->right));
}

  

void dfs_right_side(const struct TreeNode* root, const int cur_height, int *max_height, int *right_view_array, int *offset){
    if(root == NULL){
        return;
    }  

    if(cur_height > *max_height){
        *max_height = cur_height;
        right_view_array[*offset] = root->val;
        ++*offset;
    }  

    dfs_right_side(root->right, cur_height+1, max_height, right_view_array, offset);
    dfs_right_side(root->left, cur_height+1, max_height, right_view_array, offset);

}  

int* rightSideView(struct TreeNode* root, int* returnSize) {
    int tree_height = find_tree_height(root);
    int *right_view_array = (int *)malloc(tree_height*sizeof(int));
    int offset = 0;
    int max_height = 0;
    dfs_right_side(root, 1, &max_height, right_view_array, &offset);
    *returnSize = tree_height;
    return right_view_array;

}
```
## A1: 
offset相當於數組`arr[i]`中的i，所以應該dereference後再加一才合理。如果寫`*offset++` 相當於`*(offset++)`，你會移動offset在記憶體中的位置，這會錯掉。

---
## Q2 : 下面的程式在傳遞和修改字串有何問題 ?
```c
void dfs_get_smallest_string(struct TreeNode* root, char **cur_string, int *cur_string_length, char **min_string, int *min_string_length){  
    if(root == NULL){
        return;
    }    
    (*cur_string)[*cur_string_length] = 'a'+root->val;
    ++*cur_string_length;
    if(root->left == NULL && root->right == NULL && compare_string_in_reverse_order(*cur_string, *cur_string_length, *min_string, *min_string_length)){

        *min_string = *cur_string;        
        *min_string_length = *cur_string_length;
    }

  

    dfs_get_smallest_string(root->left, cur_string, cur_string_length, min_string, min_string_length);
    dfs_get_smallest_string(root->right, cur_string, cur_string_length, min_string, min_string_length);
    --*cur_string_length;
    (*cur_string)[*cur_string_length] = '\0';
}

char* smallestFromLeaf(struct TreeNode* root) {
    if(root == NULL)    return NULL;
    int tree_height = get_tree_height(root);
    char *cur_string = calloc(tree_height, sizeof(char));
    int cur_string_length = 0;
    char *min_string = calloc(tree_height, sizeof(char));
    int min_string_length = -1;
    dfs_get_smallest_string(root, &cur_string, &cur_string_length, &min_string, &min_string_length);  

    return min_string;
}
```
## A2 :
你在line 9只將`cur_string` 的記憶體位置指定給 `min_string`，跟本沒複製阿。你只做了淺拷貝(shallow copy)，用`strcpy()`，才會把`cur_string`的內容複製給`min_string`。