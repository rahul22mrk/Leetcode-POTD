1. 2144. Minimum Cost of buying candies with discount
2. 3633. Earliest Finish Time for Land and water rides I
3. 3635. Earliest Finish Time for Land and water rides II
4. 3751. Total Waviness of numbers in range I
5. 3753. Total Waviness of numbers in Range II
6. 2574. Left and Right Sum Differences
7. 2196. Create Binary Tree From Descriptions
8. 2161. Partition Array According to Given Pivot
9. 3689. Maximum Total Subarray Value I
10. 3691. Maximum Total Subarray Value II
11. 3558. Number of Ways to Assign Edge Weights I
   



1. 2144. Minimum Cost of buying candies with discount
2. 3633. Earliest Finish Time for Land and water rides I
3. 3635. Earliest Finish Time for Land and water rides II
4. 3751. Total Waviness of numbers in range I
5. 3753. Total Waviness of numbers in Range II
6. 2574. Left and Right Sum Differences
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
7. 2196. Create Binary Tree From Descriptions
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

8. 2161. Partition Array According to Given Pivot
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

9. 3689. Maximum Total Subarray Value I
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
10. 3691. Maximum Total Subarray Value II
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

11. 3558. Number of Ways to Assign Edge Weights I
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
