1. `2144. Minimum Cost of buying candies with discount`
2. `3633. Earliest Finish Time for Land and water rides I`
3. `3635. Earliest Finish Time for Land and water rides II`
4. `3751. Total Waviness of numbers in range I`
5. `3753. Total Waviness of numbers in Range II`
6. `2574. Left and Right Sum Differences`
7. `2196. Create Binary Tree From Descriptions`
8. `2161. Partition Array According to Given Pivot`
9. `3689. Maximum Total Subarray Value I`
10. `3691. Maximum Total Subarray Value II`
11. `3558. Number of Ways to Assign Edge Weights I`
12. `3559. Number of Ways to Assign Edge Weights II `
13. `3838. Weighted Word Mapping`
14. `2130. Maximum Twin Sum of a Linked List`

******************************** Solutions **********************************************
1. `2144. Minimum Cost of buying candies with discount`
2. `3633. Earliest Finish Time for Land and water rides I`
3. `3635. Earliest Finish Time for Land and water rides II`
4. `3751. Total Waviness of numbers in range I`
5. `3753. Total Waviness of numbers in Range II`
6. `2574. Left and Right Sum Differences`
```java
class Solution {
    public int[] leftRightDifference(int[] nums) {
        int n = nums.length;
        int ans[] = new int[n];

        int totalSum = 0;
        for(int num:nums){
            totalSum += num;
        }

        int leftSum  = 0;

        for(int i=0;i<n;i++){
            if(i>0){
                leftSum += nums[i-1];
            }

            int rightSum = totalSum - leftSum - nums[i];

            ans[i] = Math.abs(rightSum - leftSum);
        }

        return ans;
        
    }
}
```
7. `2196. Create Binary Tree From Descriptions`
```java
class Solution {
    public TreeNode createBinaryTree(int[][] descriptions) {
        
        Map<Integer, TreeNode> nodes = new HashMap<>();
        Set<Integer> children = new HashSet<>();

        for(int desc[] : descriptions){
            //current node relations 
            int parentVal = desc[0];
            int childVal = desc[1];
            int isLeft = desc[2];
            
            //create node if not exist
            nodes.putIfAbsent(parentVal, new TreeNode(parentVal));
            nodes.putIfAbsent(childVal, new TreeNode(childVal));

            TreeNode parent = nodes.get(parentVal);
            TreeNode child = nodes.get(childVal);

            if(isLeft == 1){
                parent.left = child;
            }else{
                parent.right = child;
            }

            children.add(childVal);
           
        }

        //Root = which never appeared in child
        for(int nodeVal : nodes.keySet()){
            if(!children.contains(nodeVal)){
                return nodes.get(nodeVal);
            }
        }

        return null;
    }

}
```
8. `2161. Partition Array According to Given Pivot`
```java
class Solution {
    public int[] pivotArray(int[] nums, int pivot) {
        int lessThanCnt = 0;
        int equalCnt = 0;

        for(int num : nums){
            if(num == pivot){
                equalCnt++;
            }else if(num < pivot){
                lessThanCnt++;
            }
        }

        int i = 0, j = lessThanCnt, k = j+equalCnt;
        int ans[] = new int[nums.length];

        for(int num : nums){
            if(num == pivot){
                ans[j++] = num;
            }else if(num <pivot){
                ans[i++] = num;
            }else{
                ans[k++] = num;
            }
        }
        return ans;
    }
}
```
9. `3689. Maximum Total Subarray Value I`
```java
class Solution {
    public long maxTotalValue(int[] nums, int k) {
        long max = Integer.MIN_VALUE, min = Integer.MAX_VALUE;

        for(int i=0;i<nums.length;i++){
            max = Math.max(max, nums[i]);
            min = Math.min(min,nums[i]);
        }
        return k* (max-min);
        
    }
}
```
10. `3691. Maximum Total Subarray Value II`
```java
//Approach (Segment Tree + Max Heap)
//T.C : O(n*logn + k*logn)
//S.C : O(n)

class SegmentTree {
    int[] segmentTree;
    boolean isMinTree;

    SegmentTree(int[] nums, boolean flag) {
        int n = nums.length;
        this.isMinTree = flag;
        segmentTree = new int[4 * n];
        buildSegmentTree(0, 0, n - 1, nums);
    }

    void buildSegmentTree(int i, int l, int r, int[] nums) {
        if (l == r) {
            segmentTree[i] = nums[l];
            return;
        }

        int mid = l + (r - l) / 2;

        buildSegmentTree(2 * i + 1, l, mid, nums);
        buildSegmentTree(2 * i + 2, mid + 1, r, nums);

        if (isMinTree) {
            segmentTree[i] = Math.min(segmentTree[2 * i + 1], segmentTree[2 * i + 2]);
        } else {
            segmentTree[i] = Math.max(segmentTree[2 * i + 1], segmentTree[2 * i + 2]);
        }
    }

    int querySegmentTree(int start, int end, int i, int l, int r) {
        //No overlap
        if (l > end || r < start) {
            return isMinTree ? Integer.MAX_VALUE : Integer.MIN_VALUE;
        }

        //Complete Overlap
        if (l >= start && r <= end) {
            return segmentTree[i];
        }

        int mid = l + (r - l) / 2;

        int a = querySegmentTree(start, end, 2 * i + 1, l, mid);
        int b = querySegmentTree(start, end, 2 * i + 2, mid + 1, r);

        if (isMinTree) {
            return Math.min(a, b);
        }

        return Math.max(a, b);
    }

    int query(int l, int r, int n) {
        return querySegmentTree(l, r, 0, 0, n - 1);
    }
}

class Solution {
    long getValue(int l, int r, SegmentTree minST, SegmentTree maxST, int n) {
        int minEl = minST.query(l, r, n);
        int maxEl = maxST.query(l, r, n);
        return (long) maxEl - minEl;
    }

    public long maxTotalValue(int[] nums, int k) {
        int n = nums.length;

        SegmentTree minST = new SegmentTree(nums, true);   //true is for minimum
        SegmentTree maxST = new SegmentTree(nums, false);  //false is for maximum

        //{val, l, r} max. heap
        PriorityQueue<long[]> pq = new PriorityQueue<>((a, b) -> Long.compare(b[0], a[0]));

        //Step-1 (Initialize the heap with best value)
        //O(n*logn)
        for (int l = 0; l < n; l++) {
            long value = getValue(l, n - 1, minST, maxST, n);  //log(n)
            pq.offer(new long[]{value, l, n - 1});
        }

        //Step-2 Find top k
        long result = 0;
        //O(k * log(n))
        while (k-- > 0) {
            long[] top = pq.poll();
            long value = top[0];
            int l = (int) top[1];
            int r = (int) top[2];

            result += value;

            long nextBestValue = getValue(l, r - 1, minST, maxST, n);  //log(n)

            pq.offer(new long[]{nextBestValue, l, r - 1});  //log(n)
        }

        return result;
    }
}
```
11. `3558. Number of Ways to Assign Edge Weights I`
```java
//Approach (DFS to find max depth + Fast Exponentiation)
//T.C : O(n) for DFS + O(log n) for power
//S.C : O(n) for adjacency list + O(n) recursion stack
class Solution {
    static final long MOD = 1_000_000_007L;

    private long power(long base, long exponent) {
        if (exponent == 0)
            return 1;

        long half = power(base, exponent / 2);

        long result = (half * half) % MOD;

        if (exponent % 2 == 1) {
            result = (result * base) % MOD;
        }

        return result;
    }

    private int getMaxDepth(Map<Integer, List<Integer>> adj, int node, int parent) {

        int depth = 0;

        for (int neighbor : adj.getOrDefault(node, new ArrayList<>())) {
            if (neighbor == parent)
                continue;

            depth = Math.max(depth, getMaxDepth(adj, neighbor, node) + 1);
        }

        return depth;
    }

    public int assignEdgeWeights(int[][] edges) {

        Map<Integer, List<Integer>> adj = new HashMap<>();

        for (int[] edge : edges) {
            int u = edge[0];
            int v = edge[1];

            adj.computeIfAbsent(u, k -> new ArrayList<>()).add(v);
            adj.computeIfAbsent(v, k -> new ArrayList<>()).add(u);
        }

        int maxDepth = getMaxDepth(adj, 1, 0);

        return (int) power(2, maxDepth - 1);
    }
}
```

12. `3559. Number of Ways to Assign Edge Weights II`
```java
//Approach (Using Binary Lifting): Build an ancestor table via DFS + binary lifting and answer each LCA query by lifting nodes using binary bits.
//T.C : O(N·log N + Q·log N)
//S.C : O(N·log N)
class Solution {
    int M = 1000000007;
    int n;
    int cols;

    List<List<Integer>> adj;
    int[][] ancestorTable;
    int[] depth;

    void dfs(int root, int parent) {
        ancestorTable[root][0] = parent;

        for (int ngbr : adj.get(root)) {
            if (ngbr == parent) continue;

            depth[ngbr] = depth[root] + 1;

            dfs(ngbr, root);
        }
    }

    void buildAncestorTable() {
        for (int j = 1; j < cols; j++) {
            for (int node = 0; node < n; node++) {
                if (ancestorTable[node][j - 1] != -1)
                    ancestorTable[node][j] = ancestorTable[ancestorTable[node][j - 1]][j - 1];
            }
        }
    }

    int findLCA(int u, int v) {
        if (depth[u] < depth[v]) {
            int temp = u;
            u = v;
            v = temp;
        }

        int k = depth[u] - depth[v];

        for (int j = 0; j < cols; j++) {
            if ((k & (1 << j)) != 0) {
                u = ancestorTable[u][j];
            }
        }

        if (u == v) {
            return u;
        }

        for (int j = cols - 1; j >= 0; j--) {
            if (ancestorTable[u][j] == -1) {
                continue;
            }

            if (ancestorTable[u][j] != ancestorTable[v][j]) {
                u = ancestorTable[u][j];
                v = ancestorTable[v][j];
            }
        }

        return ancestorTable[u][0];
    }

    public int[] assignEdgeWeights(int[][] edges, int[][] queries) {
        n = edges.length + 1;
        cols = (int) (Math.log(n) / Math.log(2)) + 1;

        adj = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            adj.add(new ArrayList<>());
        }

        for (int[] edge : edges) {
            int u = edge[0] - 1;
            int v = edge[1] - 1;

            adj.get(u).add(v);
            adj.get(v).add(u);
        }

        depth = new int[n];
        ancestorTable = new int[n][cols];

        for (int i = 0; i < n; i++) {
            for (int j = 0; j < cols; j++) {
                ancestorTable[i][j] = -1;
            }
        }

        dfs(0, -1);
        buildAncestorTable();

        long[] pow2 = new long[n + 1];
        pow2[0] = 1;
        for (int i = 1; i <= n; i++) {
            pow2[i] = (2L * pow2[i - 1]) % M;
        }

        int[] result = new int[queries.length];
        for (int i = 0; i < queries.length; i++) {
            int u = queries[i][0] - 1;
            int v = queries[i][1] - 1;

            int lca = findLCA(u, v);
            int d = depth[u] + depth[v] - 2 * depth[lca];

            if (d == 0) {
                result[i] = 0;
            } else {
                result[i] = (int) pow2[d - 1];
            }
        }

        return result;
    }
}
```
13. `3838. Weighted Word Mapping`
```java
class Solution {
    public String mapWordWeights(String[] words, int[] weights) {
        StringBuilder sb = new StringBuilder();

        for(String word:words){
            int sum = 0;
            for(char ch : word.toCharArray()){
                sum += weights[ch-'a'];
            }
            int temp  = 26 - 1 - (sum %26) ;
            sb.append((char) ('a'+temp));
        }

        return sb.toString();
    }
}
```
14. `2130. Maximum Twin Sum of a Linked List`
```java
Approach 1: Using ArrayList
Time: O(N)
Space: O(N)

class Solution {
    public int pairSum(ListNode head) {
        List<Integer> list = new ArrayList<>();

        while (head != null) {
            list.add(head.val);
            head = head.next;
        }

        int i = 0, j = list.size() - 1;
        int result = 0;

        while (i < j) {
            result = Math.max(result, list.get(i) + list.get(j));
            i++;
            j--;
        }

        return result;
    }
}
```

```java
Approach 2: Using Stack
Time: O(N)
Space: O(N)

class Solution {
    public int pairSum(ListNode head) {
        Stack<Integer> stack = new Stack<>();
        ListNode curr = head;

        while (curr != null) {
            stack.push(curr.val);
            curr = curr.next;
        }

        curr = head;
        int count = 1;
        int n = stack.size();
        int result = 0;

        while (count <= n / 2) {
            result = Math.max(result, curr.val + stack.pop());
            curr = curr.next;
            count++;
        }

        return result;
    }
}
```

```java
Approach 3: Reverse Second Half (Optimal)
Time: O(N)
Space: O(1)

class Solution {
    public int pairSum(ListNode head) {

        // Find middle
        ListNode slow = head;
        ListNode fast = head;

        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
        }

        // Reverse second half
        ListNode prev = null;
        ListNode curr = slow;

        while (curr != null) {
            ListNode nextNode = curr.next;
            curr.next = prev;
            prev = curr;
            curr = nextNode;
        }

        // Calculate maximum twin sum
        curr = head;
        int result = 0;

        while (prev != null) {
            result = Math.max(result, curr.val + prev.val);
            curr = curr.next;
            prev = prev.next;
        }

        return result;
    }
}
```

```java
`My Solution`
class Solution {
    public int pairSum(ListNode head) {
        ListNode midNode = middleNode(head);
        ListNode revNode = reverseList(midNode);

        ListNode curr = head;
        int maxSum = Integer.MIN_VALUE;
        while(revNode!=null){
            maxSum = Math.max(maxSum, revNode.val + curr.val);
            revNode = revNode.next;
            curr = curr.next;
        }

        return maxSum;
    }

    private ListNode middleNode(ListNode head){
        ListNode slow = head;
        ListNode fast = head;

        while(fast!= null && fast.next!=null){
            slow = slow.next;
            fast = fast.next.next;
        }

        return slow;
    }

    private ListNode reverseList(ListNode head){
        ListNode curr = head, prev = null;

        while(curr != null){
            ListNode next = curr.next;
            curr.next = prev;

            prev = curr;
            curr = next;
        }

        return prev;
    }
}
```
