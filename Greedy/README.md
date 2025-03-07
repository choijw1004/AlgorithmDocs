# Greedy
- Interval문제에서 가장 중요한 부분은 **정렬**

- 2차원 배열을 정렬하는 방법(시작점 기준)
```
Arrays.sort((a, b) -> Integer.compare(a[0], b[0]));
```
- 2차원 List를 정렬하는 방법(시작점 기준)
```
list.sort((a, b) -> Integer.compare(a.get(0), b.get(0)));
```
- 2차원 배열을 정렬하는 방법(끝점 기준)
```
Arrays.sort((a, b) -> Integer.compare(a[1], b[1]));
```
- 2차원 List를 정렬하는 방법(끝점 기준)
```
list.sort((a, b) -> Integer.compare(a.get(1), b.get(1)));
```
### 435. Non-overlapping Intervals
1. 겹치지 않는 최소 간격의 수를 return = **최대한 겹치지 않게끔 정렬을 해주고 cnt를 반환**
2. 최대한 겹치지 않게 정렬을 한다는 것은 int[] {start, end}가 있을때 end를 기준으로 정렬을 하면 되는 것
3. 정렬 후 갱신
```class Solution {
    public int eraseOverlapIntervals(int[][] intervals) {
        Arrays.sort(intervals, (a, b) -> a[1] - b[1]);
        
        int cnt = 0;
        int currEnd = Integer.MIN_VALUE;

        for(int[] interval : intervals){
            if(currEnd <= interval[0]) currEnd = interval[1];
            else cnt++;
        }
        return cnt;
    }
}
```
### 56. Merge Intervals
반대로 시작점을 기준으로 정렬하는 케이스의 문제이다 
```
import java.util.*;

class Solution {
    public int[][] merge(int[][] intervals) {
        if (intervals == null || intervals.length == 0)
            return new int[0][0];

        Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));

        List<int[]> merged = new ArrayList<>();
        
        int[] current = intervals[0];
        merged.add(current);
        
        for (int i = 1; i < intervals.length; i++) {
            int[] interval = intervals[i];
            
            if (interval[0] <= current[1]) {
                current[1] = Math.max(current[1], interval[1]);
            } else {
                current = interval;
                merged.add(current);
            }
        }
        
        return merged.toArray(new int[merged.size()][]);
    }
}
```
### 986. Interval List Intersections
두 개의 정렬된 배열에서 교집합을 만들어서 return시키는 문제이다
포인터 i, j 를 두어 while문 안에서 돌게끔 하는게 까다로워서 조건, idx들의 유연한 이동을 할 줄 알아야할 것 같다.
```
class Solution {
    public int[][] intervalIntersection(int[][] firstList, int[][] secondList) {
        List<int[]> list = new ArrayList<>();
        int i=0, j=0;
        while(i < firstList.length && j < secondList.length){
            int s1 = firstList[i][0];
            int e1 = firstList[i][1];

            int s2 = secondList[j][0];
            int e2 = secondList[j][1];

            int sMax = Math.max(s1, s2);
            int eMin = Math.min(e1,e2);

            if(sMax <= eMin) list.add(new int[] {sMax,eMin});

            if(e2 > e1) i++;
            else j++;
        }
        return list.toArray(new int[list.size()][]);
    }
}
```
### 백준 컵라면
접근부터 너무 힘들었고 자주 나올 것 같은 유형이어서 더욱 더 완벽하게 이해해야겠다는 문제였다.
시간에 관련된 문제였고 데드라인을 제시한 문제였기 때문에 데드라인 순으로 배열을 정렬해준다. 
정렬을 한 후 
컵라면 개수의 저장을 위한 pq를 하나 생성해 준 후 루프 시작.
일단 pq에 라면의 개수를 지정한다.
pq의 사이즈는 곧 해결한 task의 개수이기 때문에 
지금 데드라인을 넘어선다면 && peek(컵라면 힙의 최상위)가 현재 라면보다 작은경우 poll을 해준다.
```
package BOJ_1781_컵라면;
import java.util.*;

public class Main {
    public static int solution(int[][] problems){
        int ans = 0;

        Arrays.sort(problems, (a,b) -> Integer.compare(a[0],b[0]));

        PriorityQueue<Integer> pq = new PriorityQueue<>();

        for(int[] problem : problems){
            int dl = problem[0];
            int ramen = problem[1];

            pq.offer(ramen);

            if(pq.size() > dl && pq.peek() <= ramen) pq.poll();
        }

        while(!pq.isEmpty()) {
            ans += pq.poll();
        }
        return ans;
    }
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        int n = sc.nextInt();
        int[][] problem = new int[n][2];

        for(int i = 0 ; i < n; i++){
            problem[i][0] = sc.nextInt();
            problem[i][1] = sc.nextInt();
        }
        System.out.println(solution(problem));

    }
}

```
### BOJ_1202_보석도둑
1. input으로 배열이 두개 들어온다. 하나의 bag이 담을 수 있는 최대 무게 bags[] 보석의 무게와 가치가 주어지는 gems[]
2. 문제는 딱봐도 greedy한 문제이고 그렇다면, 어떻게 최적해를 찾을 수 있을까 고민해본다.
3. 당연히 같거나 적은 무게에 최고의 가치를 담을 수 있어야하므로 gems는 무게가 같으면 가치가 높은 순으로 
4. bags 배열 순회 시작
5. gIdx를 두어 투 포인터로 문제를 접근해보면 gIdx를 이동하면서  한계점 까지 모두 push 
6. while문이 끝나면 각 가방에 대한 poll을 통해서 가방과 보석의 매치
7. return ans; 
```
package BOJ_1202_보석도둑;
import java.util.*;

public class Main {
    public static long solution(int[][] gems, int[] bags){
        long ans = 0;
        int n = bags.length;
        int m = gems.length;
        int gIdx = 0;
        PriorityQueue<Integer> gq = new PriorityQueue<>(Collections.reverseOrder());

        //무게가 적고 가치가 높은 순
        Arrays.sort(gems, (a, b) -> {
            if(a[0] != b[0]) return Integer.compare(a[0],b[0]);
            else return Integer.compare(b[1], a[1]);
        });

        Arrays.sort(bags);

        for(int bagWeight : bags){
            while(gIdx < m && gems[gIdx][0] <= bagWeight){
                gq.offer(gems[gIdx][1]);
                gIdx++;
            }
            if(!gq.isEmpty()) ans += gq.poll();
        }
        return ans;
    }
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int n = sc.nextInt();
        int k = sc.nextInt();

        int[][] gems = new int[n][2];

        for(int i = 0 ; i < n; i++){
            gems[i][0] = sc.nextInt();
            gems[i][1] = sc.nextInt();
        }
        int[] bags = new int[k];

        for(int i = 0 ; i < k ; i++){
            bags[i] = sc.nextInt();
        }
        System.out.println(solution(gems, bags));
    }
}

```

### BOJ_1911_흙길 보수하기
1. l이라는 보수할 수 있는 cover 변수가 주어지고
2. cover할 수 있는 구간을 최대한 처음부터 겹칠 수 있게끔 처음부터 놓아야된다는 아이디어를 가지고 접근였다.

```
package BOJ_1911_흙길보수하기;
import java.util.*;

public class Main {
    public static int solution(int[][] dummys, int l){
        int ans = 0;
        int coverEnd = 0;

        Arrays.sort(dummys, (a,b) -> {
            if (a[0] == b[0]) return Integer.compare(a[1],b[1]);
            else return Integer.compare(a[0],b[0]);
        });

        for(int[] dummy : dummys){
            int s = dummy[0];
            int e = dummy[1];

            if(coverEnd < s ) coverEnd = s;

            while(coverEnd < e){
                ans++;
                coverEnd += l;
            }
        }
        return ans;
    }
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int n = sc.nextInt();
        int l = sc.nextInt();

        int[][] dummys = new int[n][2];
        for(int i = 0; i < n; i++){
            dummys[i][0] = sc.nextInt();
            dummys[i][1] = sc.nextInt();
        }
        System.out.println(solution(dummys, l));

    }
}

```
### BOJ_1508_레이스
```


```