### 1.[两数之和](https://leetcode-cn.com/problems/two-sum/)
 ```java
   public int[] twoSum(int[] nums, int target) {
        Map<Integer,Integer> hashMap = new HashMap();
        for(int i = 0;i<nums.length;i++){
            //v是需要找的那个，如果有的话，直接返回，如果没有，就把当前的存进去
            int v = target - nums[i];
            if(hashMap.containsKey(v)){
                return new int[]{i,hashMap.get(v)};
            }
            //没有配对的，把当前的存储进去
            hashMap.put(nums[i],i);
        }
        return new int[]{};
    }
 ```
 
 ### 2.
