### 三种遍历方式的特性

- 前序遍历:1 2 4 8 9 10 11 5 3 6 7  (规律：根在前；子树在根后且左子树比右子树靠前);
- 中序遍历:8 4 10 9 11 2 5 1 6 3 7  (规律：根在中；左子树在跟左边，右子树在根右边);
- 后序遍历:8 10 11 9 4 5 2 6 7 3 1  (规律：根在后；子树在根前且左子树比右子树靠前;

### 根据中序 后序结果的数组构建树

- leetcode上比较快的解决方案

```
//java
class Solution {
  int pInorder;
  int pPostorder;
  public TreeNode buildTree(int[] inorder, int[] postorder) {
    pInorder = inorder.length - 1;
    pPostorder = postorder.length - 1;
    return buildTree(inorder, postorder, null);
  }
  private TreeNode buildTree(int[] inorder, int[] postorder, TreeNode end) {
    if (pPostorder < 0) {
        return null;
    }
    // create root node;
    TreeNode n = new TreeNode(postorder[pPostorder--]);  //最后一个就是root

    // if right node exist, create right subtree
    if (inorder[pInorder] != n.val) {
        n.right = buildTree(inorder, postorder, n);
    }
    pInorder--;
    // if left node exist, create left subtree
    if ((end == null) || (inorder[pInorder] != end.val)) {
        n.left = buildTree(inorder, postorder, end);
    }
    return n;
  }
}
```