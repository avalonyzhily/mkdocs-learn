### 算法题记录
- Q: 有一个复杂链表，其结点除了有一个m_pNext指针指向下一个结点外，还有一个m_pSibling指向链表中的任一结点或者NULL。请完成函数ComplexNode Clone(ComplexNode pHead)，以复制一个复杂链表。
    - 递归+map
        ```
            public class DeepCopySingleLinked {
            private static Map<SpecialNode,SpecialNode> map = new HashMap<SpecialNode, SpecialNode>();
            public static SpecialNode copy(SpecialNode specialNode){
                if(specialNode==null){
                    return null;
                }
                SpecialNode copy = new SpecialNode();
                map.put(specialNode,copy);
                SpecialNode next = specialNode.next;
                if(next!=null){
                    if(map.containsKey(next)){
                        copy.next = map.get(next);
                    }else {
                        SpecialNode copyNext = new SpecialNode(specialNode.val);
                        map.put(next,copyNext);
                        copyNext = copy(next);
                        copy.next = copyNext;
                    }
                }
                SpecialNode extra = specialNode.extra;
                if(extra!=null){
                    copy.extra = map.get(extra);
                }
                copy.val = specialNode.val;
                return copy;
            }

            public static void main(String[] args){
                SpecialNode node1 = new SpecialNode(1);
                SpecialNode node2 = new SpecialNode(2);
                SpecialNode node3 = new SpecialNode(3);
                SpecialNode node4 = new SpecialNode(4);
                SpecialNode node5 = new SpecialNode(5);
                SpecialNode node6 = new SpecialNode(6);

                node1.next = node2;
                node2.next = node3;
                node3.next = node4;
                node4.next = node5;
                node5.next = node6;

                node1.extra = node2;
                node2.extra = node5;
                node3.extra = node1;
                node4.extra = node6;

                SpecialNode res = copy(node1);
                System.out.println(res);
            }
        }
        public class SpecialNode {
            public SpecialNode next;
            public int val;
            public SpecialNode extra;

            public SpecialNode() {
            }

            public SpecialNode(int val) {
                this.val = val;
            }
        }
        ```
    - 遍历+map
        ```
            public static SpecialNode copy(SpecialNode specialNode){
                Map<SpecialNode,SpecialNode> map = new HashMap<SpecialNode, SpecialNode>();
                if(specialNode==null){
                    return null;
                }
                SpecialNode origin = specialNode;
                while (origin!=null){
                    SpecialNode temp = new SpecialNode(origin.val);
                    map.put(origin,temp);
                    origin = temp.next;
                }
                origin = specialNode;
                while (origin!=null){
                    SpecialNode temp = map.get(origin);
                    temp.next = map.get(origin.next);
                    temp.extra = map.get(origin.extra);
                    origin = origin.next;
                }
                return map.get(specialNode);
            }

            public static void main(String[] args){
                SpecialNode node1 = new SpecialNode(1);
                SpecialNode node2 = new SpecialNode(2);
                SpecialNode node3 = new SpecialNode(3);
                SpecialNode node4 = new SpecialNode(4);
                SpecialNode node5 = new SpecialNode(5);
                SpecialNode node6 = new SpecialNode(6);

                node1.next = node2;
                node2.next = node3;
                node3.next = node4;
                node4.next = node5;
                node5.next = node6;

                node1.extra = node2;
                node2.extra = node5;
                node3.extra = node1;
                node4.extra = node6;

                SpecialNode res = copy(node1);
                System.out.println(res);
            }
            public class SpecialNode {
            public SpecialNode next;
            public int val;
            public SpecialNode extra;

            public SpecialNode() {
            }

            public SpecialNode(int val) {
                this.val = val;
            }
        }
        ```
    - 遍历
    ```
        public static SpecialNode copy(SpecialNode specialNode){
        if(specialNode==null){
            return null;
        }
        SpecialNode origin = specialNode;
        SpecialNode temp = null;
        while (origin!=null){
            temp = origin.next;
            SpecialNode newNode = new SpecialNode(origin.val);
            newNode.next = temp;
            origin.next = newNode;
            origin = origin.next;
        }
        origin = specialNode;
        while (origin!=null){
            origin.next.extra = origin.extra!=null?origin.extra.next:null;
            origin = origin.next!=null?origin.next.next:origin.next;
        }
        origin = specialNode;
        SpecialNode copyNode = origin.next;
        temp = copyNode;
        while (origin!=null && copyNode !=null){
            origin.next = origin.next!=null?origin.next.next:origin.next;
            copyNode.next = copyNode.next!=null?copyNode.next.next:copyNode.next;
            origin = origin.next;
            copyNode = copyNode.next;
        }
        return temp;
    }

    public static void main(String[] args){
        SpecialNode node1 = new SpecialNode(1);
        SpecialNode node2 = new SpecialNode(2);
        SpecialNode node3 = new SpecialNode(3);
        SpecialNode node4 = new SpecialNode(4);
        SpecialNode node5 = new SpecialNode(5);
        SpecialNode node6 = new SpecialNode(6);

        node1.next = node2;
        node2.next = node3;
        node3.next = node4;
        node4.next = node5;
        node5.next = node6;

        node1.extra = node2;
        node2.extra = node5;
        node3.extra = node1;
        node4.extra = node6;

        SpecialNode res = copy(node1);
        System.out.println(res);
    }
    public class SpecialNode {
            public SpecialNode next;
            public int val;
            public SpecialNode extra;

            public SpecialNode() {
            }

            public SpecialNode(int val) {
                this.val = val;
            }
        }
    ```