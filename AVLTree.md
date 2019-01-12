二叉平衡树

[TOC]

# 定义
平衡二叉树，又称AVL树，它是一种特殊的二叉排序树。AVL树或者是一棵空树，或者是具有以下性质的二叉树：
1. 左子树和右子树都是平衡二叉树；
2. 左子树和右子树的深度（高度）之差的绝对值不超过1。

# 结点定义

```java
public class TreeNode {
    TreeNode _leftNode;
    TreeNode _rightNode;
    NodeData _nodeData;
    int _key;
    int _height;

    TreeNode(int key, TreeNode left, TreeNode right, NodeData data) {
        _key = key;
        _leftNode = left;
        _rightNode = right;
        _nodeData = data;
        _height = 1;
    }

}
```

# 操作
有遍历（先序、中序、后序），最大值与最小值，插入和删除，销毁二叉树等操作。
## 遍历

### 先序遍历
```java
    public void preOrder(TreeNode root) {
        if (root != null) {
            System.out.println("root key = " + root._key);
            preOrder(root._leftNode);
            preOrder(root._rightNode);
        }
    }
```
### 中序遍历
```java
    public void inOrder(TreeNode root) {
        if (root != null) {
            inOrder(root._leftNode);
            System.out.println("root key = " + root._key);
            inOrder(root._rightNode);
        }
    }
```
### 后序遍历
```java
    public void postOrder(TreeNode root) {
        if (root != null) {
            postOrder(root._leftNode);
            postOrder(root._rightNode);
            System.out.println("root key = " + root._key);
        }
    }
```

## 旋转
在进行插入和删除之前需要先了解AVL树的旋转操作。旋转操作主要包括LL（左左）旋转、LR（左右）旋转、RR（右右）旋转、RL（右左）旋转，LL旋转与RR旋转对称，LR旋转与RL旋转对称。旋转操作是在插入结点或删除结点导致原AVL树不平衡时进行的。个人理解是当二叉树失衡的原因出现在"最低失衡根结点左子树的左子树"（所谓"最低失衡根结点"，**是从新增结点开始向根部回溯，所遇到的第一个失衡的根节点**）时，则使用LL旋转来调整；当失衡出现在“最低失衡根节点左子树的右子树”，则使用LR旋转调整；RR旋转，RL旋转同理。
### 左左旋转
![左左旋转前](https://images2017.cnblogs.com/blog/1258764/201711/1258764-20171105103356076-23076749.png)
如上图所示，找到“最低失衡根结点”，上图是结点5，二叉树失衡的原因是因为结点1的存在，而结点1位于结点5“左子树的左子树”，所以要使用LL旋转来调节，只需一次旋转即可达到平衡。具体的方法是：LL旋转的对象是“最低失衡根结点”，也就是结点5，找到5的左孩子3，将3的右孩子4变成5的左孩子，最后将5变成3的右孩子，调整后的AVL树如下所示：
![左左旋转后](https://images2017.cnblogs.com/blog/1258764/201711/1258764-20171105103845685-1592716257.png)

```java
    /**
     * root左孩子的左孩子插入节点后，root失去平衡
     * @param root 旋转点-----最低失衡点,调整之后的根节点
     * @return
     */
    private TreeNode leftLeftRotate(TreeNode root) {
        TreeNode left = root._leftNode;
        root._leftNode = left._rightNode;
        left._rightNode = root;
        root._height = max(height(root._leftNode), height(root._rightNode)) + 1;
        left._height = max(height(left._leftNode), height(left._rightNode)) + 1;
        return left;
    }
```
### 右右旋转
![](https://images2017.cnblogs.com/blog/1258764/201711/1258764-20171105104908529-2009887250.png)
如上图所示，“最低失衡根结点”是结点2，二叉树的失衡是结点6导致的，而结点6位于结点2“右子树的右子树”，所以要使用RR旋转来调节，只需一次旋转即可达到平衡。方法与LL旋转类似：RR旋转的对象是“最低失衡根结点”，这里是结点2，找到2的右孩子4，将4的左孩子3变成2的右孩子，最后将2变成4的右孩子，旋转后的结果如下图所示：
![](https://images2017.cnblogs.com/blog/1258764/201711/1258764-20171105105605732-69984508.png)
```java
    /**
     * root右孩子的右孩子插入节点后，root失去平衡
     * @param root 旋转点-----最低失衡点,调整之后的根节点
     * @return
     */
    private TreeNode rightRightRotate(TreeNode root) {
        TreeNode right = root._rightNode;
        root._rightNode = right._leftNode;
        right._leftNode = root;
        root._height = max(height(root._leftNode), height(root._rightNode)) + 1;
        right._height = max(height(right._leftNode), height(right._rightNode)) + 1;
        return right;
    }
```
### 左右旋转
LL旋转和RR旋转只需一次旋转即可达到平衡，而LR旋转和RL旋转需两次旋转才能达到平衡。
![](https://images2017.cnblogs.com/blog/1258764/201711/1258764-20171105150008123-771599762.png)
 如上图所示，“最低失衡根结点”为结点5，二叉树失衡是因为结点3的存在，结点3位于结点5“左子树的右子树”，所以使用LR旋转来调节。方法：（1）先对最低失衡根结点的左孩子（结点2）进行RR旋转；（2）再对最低失衡根结点（结点5）进行LL旋转。下图演示了调整过程。
 ![](https://images2017.cnblogs.com/blog/1258764/201711/1258764-20171105153630810-1833692477.png)
```java
    /**
     * root左孩子的右孩子插入节点后，root失去平衡
     * @param root
     * @return
     */
    private TreeNode leftRightRotate(TreeNode root) {
        rightRightRotate(root._leftNode);
        return leftLeftRotate(root);
    }
```
### 右左旋转
RL旋转与LR旋转对称，先LL旋转，在RR旋转。
![](https://images2017.cnblogs.com/blog/1258764/201711/1258764-20171105154058607-92808324.png)
分析过程与LR相似。旋转步骤：（1）先对最低失衡结点右孩子（结点5）LL旋转；（2）在对最低失衡结点（结点2）RR旋转。旋转过程如下：
![](https://images2017.cnblogs.com/blog/1258764/201711/1258764-20171105154730107-1827835.png)
```java
    /**
     * root右孩子的左孩子插入节点后，root失去平衡
     * @param root
     * @return
     */
    private TreeNode rightLeftRotate(TreeNode root) {
        leftLeftRotate(root._rightNode);
        return rightRightRotate(root);
    }
```

## 插入(insert)

```java
    public TreeNode insert(TreeNode root, int key) {
        if (root == null) {
            return new TreeNode(key, null, null, null);
        } else if (root._key > key) {
            root._leftNode = insert(root._leftNode, key);
            int delta = height(root._leftNode) - height(root._rightNode);
            if (delta >= 2 || delta <= -2) {
                if (root._leftNode._key > key) {
                    root = leftLeftRotate(root);
                } else {
                    root = leftRightRotate(root);
                }
            }
        } else if (root._key < key){
            root._rightNode = insert(root._rightNode, key);
            int delta = height(root._rightNode) - height(root._leftNode);
            if (delta >= 2 || delta <= -2) {
                if (root._rightNode._key < key) {
                    root = rightRightRotate(root);
                } else {
                    root = rightLeftRotate(root);
                }
            }
        }
        root._height = max(height(root._leftNode), height(root._rightNode)) + 1;
        return root;
    }
```

## 删除(delete)