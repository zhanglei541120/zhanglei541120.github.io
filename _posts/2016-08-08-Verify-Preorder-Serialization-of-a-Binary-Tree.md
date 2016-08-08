One way to serialize a binary tree is to use pre-order traversal. When we encounter a non-null node, we record the node's value. If it is a null node, we record using a sentinel value such as <b>#</b>.

    """    _9_
          /   \
         3     2
        / \   / \
       4   1  #  6
      / \ / \   / \
      # # # #   # # """

For example, the above binary tree can be serialized to the string <code>"9,3,4,#,#,1,#,#,2,#,6,#,#"</code>, where # represents a null node.

Given a string of comma separated values, verify whether it is a correct preorder traversal serialization of a binary tree. Find an algorithm without reconstructing the tree.

Each comma separated value in the string must be either an integer or a character '#' representing null pointer.

You may assume that the input format is always valid, for example it could never contain two consecutive commas such as <code>"1,,3"</code>.


    Example 1:
    <code>"9,3,4,#,#,1,#,#,2,#,6,#,#"</code>
    Return true

    Example 2:
    <code>"1,#"</code>
    Return false

    Example 3:
    <code>"9,#,#,1"</code>
    Return false