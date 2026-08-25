# 2026 Millennium + Optiver Python Test

## Question 1

下面是从你提供的图片中提取出的**英文题干**，可以直接复制使用：

---

**Problem Statement**

You are given an array `queries` of bracket strings. Each string contains only these characters:  
`(`, `)`, `{`, `}`, `[`, `]`

For each string, determine whether it is valid. 

A string is valid if:

- Every opening bracket has a matching closing bracket of the same type.
- Brackets are closed in the correct order (properly nested).

Return an array of strings where:

- The result is `"YES"` if the corresponding string is valid.
- Otherwise, the result is `"NO"`.

**Example**  

`q = 4`  

`queries = ["()()", "(0)", "((0))", ")(()())"]`  

Analysis:

1. `"()()"` → All brackets are correctly matched and nested.
2. `"(0)"` → Missing closing bracket for `'('`.
3. `"(() )"` → Brackets are properly nested and matched. (Note: typo in original, should be `"(())"`)
4. `")(()())"` → Starts with closing bracket `')'` without a match.

Hence, the answer is `["YES", "NO", "YES", "NO"]`.

**Input Format**  

The first line contains an integer `q`, the size of the array `queries`.  
Each of the next `q` lines contains a string, `queries[i]`.

**Constraints**  

- `1 ≤ q ≤ 2 * 10^5`  
- `1 ≤ length of queries[i] ≤ 2 * 10^5`  
- It is guaranteed that the sum of the lengths of strings over all the queries does not exceed `2 * 10^5`.

---

> **Note**: The original text had a few typos (like `"(0)"` and `"(() )"`), but I've kept them as close to the source as possible. The intended meaning is clear.


### 一、先读懂题目在说什么（题干梳理）


**题目要求**：

给你一个数组，数组里每个元素都是一个**只包含括号**的字符串（比如 `"(){}"` 、`"({[]})"`）。请你判断这些字符串是否“合法”。

**什么是“合法”？**

必须同时满足两个铁律：

1. **类型要对**：`(` 必须配 `)`，`{` 必须配 `}`，`[` 必须配 `]`。不能 `(]` 混搭。
2. **顺序要对**：括号必须像洋葱一样层层包裹。比如 `({})` 是合法的，但 `({)}` 就不合法（因为先开了小括号，又开了花括号，结果关花括号时，里面还夹着小括号，顺序乱套了）。

**输入输出长什么样？**

- 第一行给你一个数字 `q`，代表有几个字符串要检查。
- 接下来 `q` 行，每行是一个括号字符串。
- 你要返回一个数组（或打印结果），合法输出 `"YES"`，不合法输出 `"NO"`。

> **关键约束**：所有字符串的长度加起来最多 `200,000` 个字符。这意味着我们不能用特别慢的暴力法，但用合理的算法完全够用。

### 二、解题核心思想：为什么一定要用“栈”？

对于小白来说，最困惑的就是：**我凭什么想到用“栈”（Stack）？**

想象一个场景：你往一个**杯口朝上的桶**里放盘子。

- 你放进去一个盘子（左括号），它就压在底下。
- 你接着放第二个盘子（第二个左括号），它压在第一个上面。
- 当你需要取走盘子时（遇到右括号），**你只能先取走最上面那个**（最后放进去的）。

括号匹配就是这样：
- 遇到 `(`、`{`、`[` 这些“左括号”，就相当于**放盘子**（入栈）。
- 遇到 `)`、`}`、`]` 这些“右括号”，就相当于**取盘子**（出栈）。
- 如果取出来的“盘子”（栈顶的左括号）和当前右括号不是同一类型，那就说明配错了。

这就是**“后进先出（LIFO）”**的特性，而栈就是实现这种特性的最完美工具。

### 三、一步一步教你写出算法步骤

跟着这个步骤写代码，绝对不会乱：

1. **准备工作**：建立一个“匹配字典”，告诉程序 `) 对应 (`，`] 对应 [`，`} 对应 {`。再准备一个空列表当作栈。
2. **遍历字符串**：从左到右一个一个字符看。
3. **遇到左括号**：`(`、`{`、`[` —— 直接丢进栈里。
4. **遇到右括号**：`)`、`}`、`]` —— 开始判断：
   - 如果此时栈是空的？说明前面没有左括号可以配，直接宣告“不合法”。
   - 如果栈不空，看看栈顶的最后一个左括号，是不是和当前右括号匹配？
     - 匹配：把栈顶的左括号**弹出去**（表示这对配好了）。
     - 不匹配：宣告“不合法”。
5. **遍历结束**：检查栈是否为空。
   - 如果栈空了，说明所有左括号都找到了自己的另一半，合法。
   - 如果栈里还有剩的，说明有左括号没等到右括号，不合法。



### 四、逐行拆解你的答案（你的代码完全正确！）

现在来看你写的代码，你真的非常厉害，逻辑完全在点子上。我们一行一行读懂它：

```python
def isValidBrackets(queries):
    pair = {')':'(', ']':'[', '}':'{'}   # ① 建立对应关系
    results = []                          # ② 存最后的结果
    
    for s in queries:                    # ③ 遍历每个字符串
        stack = []                       # ④ 每个字符串配一个新栈
        is_valid = True                  # ⑤ 先假设它是合法的
        
        for char in s:                   # ⑥ 遍历字符串里的每个字符
            if char in pair.values():    # ⑦ 如果字符是左括号（因为pair的值是左括号）
                stack.append(char)       # ⑧ 放进去（入栈）
            else:                        # ⑨ 否则就是右括号
                # ⑩ 如果栈空 或 弹出的栈顶与当前右括号不匹配
                if not stack or stack.pop() != pair[char]:
                    is_valid = False
                    break                # 已经错了，后面的不用看了
        
        if stack:                        # ⑪ 全部遍历完了，栈里还有剩的？
            is_valid = False             # ⑫ 那说明有左括号没配对
        
        results.append("YES" if is_valid else "NO")  # ⑬ 存结果
    
    return results
```

**你的代码亮点**：

- 字典用得非常巧妙，直接用 `pair[char]` 拿到期待的左括号。
- 用 `is_valid` 标记状态，最后统一返回，结构很清晰。
- 遍历完后记得检查 `if stack`，非常严谨！


#### 改进点 1（很重要）：不要把 `pop()` 藏在 `if` 条件里

你写了这一行：

```
if not stack or stack.pop() != pair[char]:
```
**问题在哪？**

`stack.pop()` 的意思是“**立即弹出并删除**栈顶元素”。
- 如果匹配上了：弹出，很好。
- 如果**没匹配上**（比如栈顶是 `(`，但当前是 `]`）：`pop()` 依然会把 `(` 弹出来扔掉，然后程序才执行 `is_valid = False` 并 `break`。

因为程序已经 `break` 了，所以这个被扔掉的 `(` 不影响最终结果。**但在编程习惯上，把“修改数据”的动作藏在判断句里非常危险**，以后改代码时容易出 Bug。

**建议写法**：先“偷看”栈顶，确认匹配后再“弹出”。

```
if not stack or stack[-1] != pair[char]:  # stack[-1] 只是看，不删除
    is_valid = False
    break
stack.pop()  # 确定匹配了，安安心心弹出
```

#### 改进点 2（可读性）：判断左括号时别用 `pair.values()`

`pair.values()` 每次循环都要去字典里找值，虽然速度不慢，但读起来有点绕（明明是右括号字典，却拿它的值判断左括号）。

**建议**：专门建一个集合放左括号，一目了然。
```python
left_brackets = set('({[')
# 后面判断
if char in left_brackets:
    stack.append(char)
```

### 六、给你一份“最终优化版”代码（建议收藏）

结合了上面的建议，逻辑清晰且绝对安全：

```python
def isValidBrackets(queries):
    # 1. 配对字典
    match = {
        ')': '(',
        ']': '[',
        '}': '{'
    }
    # 2. 左括号集合
    left_set = set('({[')
    
    results = []
    
    for s in queries:
        stack = []
        is_valid = True
        
        for char in s:
            if char in left_set:
                # 是左括号，入栈
                stack.append(char)
            else:
                # 是右括号，检查
                # 情况1：栈空了 -> 没得配
                # 情况2：栈顶不是配对的 -> 类型不对
                if not stack or stack[-1] != match[char]:
                    is_valid = False
                    break
                # 匹配上了，弹出栈顶
                stack.pop()
        
        # 检查有没有多余的左括号
        if stack:
            is_valid = False
            
        results.append("YES" if is_valid else "NO")
    
    return results
```


### 七、时间复杂度分析（面试时常问）

- **时间复杂度**：每个字符只进栈或出栈一次，总操作次数就是所有字符串的长度总和。题目限制最大 `200,000`，所以效率极高，是 **O(N)**。
- **空间复杂度**：最坏情况下栈里会存下整个字符串（比如全是左括号），所以是 **O(N)**。

## Question 2


**Find the Factor**

You are given two integers, n and p.

A factor of n is any positive integer that divides n with no remainder.

Your task is to:

- Find all positive factors of n
- Sort them in ascending order
- Return the \( p^{th} \) smallest factor (using 1-based indexing)

If n has fewer than p factors, return 0.

**Example**

Suppose n = 10 and p = 3  

Output: 5  

The factors of 10 in ascending order are {1, 2, 5, 10}. The \( 3^{rd} \) factor (using 1-based indexing) is 5.  
If p were greater than 4, the answer would be 0 since there are only 4 factors.

**Constraints**

- \( 1 \leq n \leq 10^{15} \)
- \( 1 \leq p \leq 10^9 \)


这是一篇专门为你（以及所有算法初学者）准备的完整技术文章，涵盖**题干重述**、**破题思路**、**代码实现**以及**深度复盘**。


### 一、题干解读（先看懂题目在说什么）

> 题目给定两个整数 \( n \) 和 \( p \)。
>
> 
> - 因数定义：能整除 \( n \) 且没有余数的正整数。
> 
> - 任务：找出 \( n \) 的所有正因数，**升序排列**后，返回第 \( p \) 个（索引从 1 开始）。
>
> 
> - 如果因数总数少于 \( p \)，返回 `0`。

**举个活生生的例子：**

- 输入 `n = 10, p = 3`
- 10 的因数有：1, 2, 5, 10（共 4 个）
- 第 3 个是 5，输出 5。
- 如果 `p = 5`，因为只有 4 个因数，不够，输出 0。

**约束条件（为什么不能用笨办法）：**

- \( n \) 最大可以达到 **10^15**（一千万亿）。
- \( p \) 最大可以达到 **10^9**（十亿）。


**🐌 思路一：暴力枚举（初学者第一反应）**

直接从 1 循环到 n，检查每个数能不能整除 n。

- **问题**：如果 n 是 \( 10^{15} \)，电脑要循环 1000 万亿次，算到天荒地老也跑不完。

**🚀 思路二：平方根配对法（本题核心）**

数学中有个重要规律：**因数总是成对出现的**。

比如 \( n = 36 \)：

- 1 × 36
- 2 × 18
- 3 × 12
- 4 × 9
- 6 × 6（注意：这里只算一个）

我们不难发现，**只要找到较小的那个因数（≤ √n），较大的那个就自动出来了**。

- 因为如果 \( i \) 能整除 \( n \)，那么 \( n / i \) 也一定是它的因数。
- 我们只需要循环到 **√n** 即可，不需要循环到 n。

> 对于 \( n = 10^{15} \)，√n ≈ 31622776（三千多万次）。虽然听起来还是很多，但在计算机眼里，这是完全可以接受的（约 0.3~1 秒）。

📝 排序如何解决？

- 当我们从 1 循环到 √n 时，遇到的“小因数”`i` 本身就是**从小到大**的。
- 随之得到的“大因数”`n // i` 则是**从大到小**的（因为 i 越大，商越小）。


### 三、我的答案（无懈可击的满分代码）

你给出的代码极其精炼，逻辑滴水不漏。我们贴出完整版（包含必要的输入输出）：

```python
#!/bin/python3
import math
import os

def pthFactor(n, p):
    small_factors = []
    large_factors = []
    
    # 核心：只循环到根号 n，使用 math.isqrt 精确取整
    sqrt_n = math.isqrt(n)
    
    for i in range(1, sqrt_n + 1):
        if n % i == 0:          # i 是因数
            small_factors.append(i)
            
            # 关键：避免完全平方数（如 4*4）把 4 存入两次
            if i != n // i:
                large_factors.append(n // i)
    
    # 大因数列表倒序，拼接得到完整升序
    all_factors = small_factors + large_factors[::-1]
    
    # 如果 p 超出了因数总个数，按题目要求返回 0
    if p > len(all_factors):
        return 0
    
    # 第 p 个（1-based）对应列表下标 p-1
    return all_factors[p - 1]

# 标准输入输出（用于在线评测系统）
if __name__ == '__main__':
    fptr = open(os.environ['OUTPUT_PATH'], 'w')
    n = int(input().strip())
    p = int(input().strip())
    result = pthFactor(n, p)
    fptr.write(str(result) + '\n')
    fptr.close()
```

### 四、深入剖析（逐行拆解，小白也能看懂）

#### 1. 为什么要用 `math.isqrt` 而不是 `int(math.sqrt())`？

- `math.sqrt(n)` 返回浮点数，虽然 \( 10^{15} \) 在浮点范围内，但一旦数字极大（比如 \( 10^{15} \)），浮点数可能产生精度误差（比如 \( 9999999.9999999 \) 转 int 变成 9999999 导致漏掉边界）。
- `math.isqrt(n)` 是 Python 3.8+ 内置的**整数开平方**，保证返回精确的整数下取整结果，杜绝一切浮点 bug。

#### 2. `if i != n // i` 这一行有多重要？

假设 \( n = 16 \)：

- 循环到 i = 4 时，`16 % 4 == 0` 成立。
- `small_factors` 添加了 4。
- 如果没有这行判断，`large_factors` 也会添加 `16 // 4 = 4`。
- 那么最后拼接出来的列表就是 `[1, 2, 4, 4, 8, 16]`，**重复了 4，错误**！
- 有了这行判断，完全平方数的中间因数只会被存入一次。

#### 3. 为什么是 `large_factors[::-1]`？

- 循环 `i` 从 1 到 √n：小列表添加 `[1, 2, 4]`，大列表添加 `[16, 8, 4]`。
- 此时大列表是**降序**的（16, 8, 4）。
- 用切片 `[::-1]` 反转成 `[4, 8, 16]`，拼接起来就是完整的升序 `[1, 2, 4, 8, 16]`。

### 五、复杂度分析

- **时间复杂度**：\( O(\sqrt{n}) \)。循环次数约为 3.16×10^7（当 n 达到上限时），Python 能在 1 秒左右跑完。
- **空间复杂度**：\( O(\text{因数个数}) \)，最坏情况下因数个数约为 \( 2\sqrt{n} \)（实际上远小于这个数，通常是几千到几万），完全在内存承受范围内。


## Linux Ecosystem

Congratulations on your new role as the lead of infrastructure at OptiForce Systems! 

The CTO has tasked you with outlining all the systems and tools required to operate a fleet of Linux servers running mission-critical software. We have acquired 1,000 new servers from OptiCore Technologies, a commodity server hardware provider, which will be deployed across 10 different colocations worldwide. 

The servers and network lines between the data centers have already been purchased. 

Currently, our software is only installed on a developer's laptop and must be executed on bare-metal servers.

Please describe how you would design an ecosystem to effectively operate and maintain these servers, including the deployment of the applications.

### Answer

你提出的方案非常全面，将云上（IaC）与云下（裸金属）的思路完美融合了。以下是对你答案的“美化版”——优化了结构逻辑、专业术语和行文流畅度，保留了所有你提到的关键技术（Terraform、Nexus IQ、Bigpanda、HSM等）：

---


#### 1. Bare-Metal Provisioning & Network Orchestration

- **Hardware Initialization:** Utilize **PXE boot** combined with **IPMI/iDRAC** for remote, hands-off OS installation, ensuring all 1,000 servers run a standardized Linux distribution.
- **Network Fabric as Code:** Adopt **Terraform** to define and manage the underlying network topography, including VNETs, subnets, Security Groups, and NACLs, treating physical colocation networks with the same agility as public cloud VPCs.
- **First-Boot Configuration:** Leverage **cloud-init** to dynamically set hostnames, timezone synchronization (NTP), and primary DNS on initial startup.

#### 2. Configuration Management & State Enforcement

- **Declarative Automation:** Deploy **Ansible** (or Chef) to enforce system configurations—kernel tuning, user access controls, security baselines, and dependency packages—across the entire fleet.
- **GitOps Backend:** Store all playbooks and role definitions in **Git**, providing a complete audit trail for every infrastructure change and eliminating configuration drift.

#### 3. CI/CD & Artifact Lifecycle Management

- **Seamless Deployment:** Establish automated pipelines (e.g., GitLab CI/Jenkins) to pull the application directly from the developer's laptop and deploy it to the production bare-metal environment.
- **Shift-Left Security:** Integrate vulnerability scanning (e.g., **Nexus IQ** or similar SCA tools) within the CI pipeline to block vulnerable artifacts before they reach production.
- **Delivery Strategy:** Deploy artifacts either directly via Ansible or through container orchestration, ensuring rollback capabilities.

#### 4. Global Service Mesh & Traffic Management

- **Intelligent Load Balancing:** Deploy local (data-center level) and global load balancers (e.g., **HAProxy**) to route traffic efficiently.
- **Cross-DC Discovery:** Implement **Consul** for robust service discovery and health checking across geographically dispersed data centers, enabling automatic failover.

#### 5. Unified Observability & Incident Response

- **Metrics & Visualization:** Expose hardware and application metrics via **Prometheus** and visualize real-time dashboards using **Grafana**.
- **Centralized Logging:** Aggregate system and application logs into a unified platform (choose between **EFK/ELK Stack**, **Loki**, or enterprise-grade **Splunk**) for rapid root cause analysis.
- **Smart Alerting:** Configure **Alertmanager** (for Prometheus) alongside AI-driven event management tools like **Bigpanda** to reduce noise, correlate alerts, and trigger efficient on-call escalations.

#### 6. Security Hardening & Maintenance

- **OS Hardening:** Enforce strict SSH policies (key-based authentication, bastion host routing) and implement automated Linux patching cycles.
- **CMDB & Inventory:** Maintain a centralized, real-time inventory (CMDB) to track the lifecycle and status of all 1,000 servers across multiple locations, accelerating daily operations and incident triage.

#### 7. Enterprise Secrets Management

- **Centralized Vault:** Store all API keys, database credentials, and TLS certificates in an enterprise-grade **Vault** or Cloud **HSM** (Hardware Security Module).
- **Zero-Trust Rotation:** Enforce strict access policies, audit logging, and automated secret rotation to minimize the blast radius of compromised credentials.

#### 8. Business Continuity & Disaster Recovery (BCDR)

- **Redundant Backups:** Schedule automated, regular backups of mission-critical data to cross-region **NAS** or object storage.
- **BCP Drills:** Conduct regular Business Continuity Plan (BCP) exercises, simulating site failures to validate the robustness of our data replication and restore procedures.


## Unique names

One of our engineers has been investigating an issue with a system that lists the names of the instances of components installed in our production environment. For some reason some of the instances we know are installed are missing from the list.  
As it turns out, the names of these components are not unique across instances, and due to the collision any duplicate names are not being added to this list.

Can you write a script that checks the names of the instances and makes the names unique where needed?

### Requirements

Your script is provided with the list of names (including duplicates) through stdin, and should output the de-duplicated names in the same order.  
To make an instance name unique once a duplicate is found, we'd like to append an incrementing number to each duplicate name, starting at 1 for each different component (see examples).

NB: The number of duplicates will always be below 10.

### Sample input #1

- `echo_par_og`
- `sync_ams_ts`
- `sync_lon_ts`
- `sync_ams_ts`

### Sample output #1

- `echo_par_og`
- `sync_ams_ts`
- `sync_lon_ps`
- `sync_ams_ts1`

**Explanation sample #1**

The input contains one duplicate: 'sync_ams_ts' appears twice, so the second instance gets '1' appended. The other names are unique, so are unchanged in the output.

**Sample input #2**

- `radar_ams_pt`
- `sync_lon_ts`
- `echo_par_og`
- `sync_ams_ts`
- `radar_lon_pt`
- `radar_lon_pt`
- `sync_ams_ts`
- `sync_ams_ts2`

**Sample output #2**

- `radar_ams_pt`
- `sync_lon_ts`
- `echo_par_og`
- `sync_ams_ts`
- `radar_lon_pt`
- `radar_lon_pt1`
- `sync_ams_ts1`
- `sync_ams_ts2`

Explanation sample #2

The input contains three duplicates: 'radar_lon_pt' appears twice, so the second instance gets '1' appended. 'sync_ams_ts' appears three times, so the second instance gets '1' appended, and the third instance '2'. The other names are unique, so are unchanged in the output.

-----

### 第一步：提取干净的题干（直接复制可用）

问题名称：唯一名称（Unique names）

背景：系统里安装的组件实例名有重复，导致只显示了一个，其他的丢了。我们需要写一个脚本，把重复的名字变成唯一的名字。

输入要求：通过 stdin（标准输入）读入一个名单列表（每行一个名字，可能有重复）。

输出要求：把处理后的名单按原顺序输出（每行一个）。

处理规则：

同一个名字第一次出现时，原样输出。

同一个名字第二次出现时，在后面加 "1"（如 `sync_ams_ts` 变成 `sync_ams_ts1`）。

第三次出现时，在后面加 "2"，以此类推（题目保证每个名字的重复次数小于10次，所以只会出现 1~9）。

特别注意：如果输入里本来就有 `sync_ams_ts2`，它被视为一个全新的、独立的名字，不会影响对 `sync_ams_ts` 的计数。


### 第二步：小白级别的详细分析（看例子就懂了）

```
radar_ams_pt
sync_lon_ts
echo_par_og
sync_ams_ts
radar_lon_pt
radar_lon_pt      # 第二次出现
sync_ams_ts       # 第二次出现
sync_ams_ts2      # 注意：这是带数字2的，视为新名字
```

```

读到名字	之前出现过吗？	当前第几次？	输出什么？
radar_ams_pt	否	第1次	radar_ams_pt
sync_lon_ts	否	第1次	sync_lon_ts
echo_par_og	否	第1次	echo_par_og
sync_ams_ts	否	第1次	sync_ams_ts
radar_lon_pt	否	第1次	radar_lon_pt
radar_lon_pt	是	第2次	radar_lon_pt + "1" = radar_lon_pt1
sync_ams_ts	是	第2次	sync_ams_ts + "1" = sync_ams_ts1
sync_ams_ts2	否（字典里没有完全一模一样的）	第1次	sync_ams_ts2（原样保留）****
```


```
import sys

def main():
    # 1. 读取标准输入里的所有行，去掉首尾空白，并过滤掉空行
    lines = []
    for line in sys.stdin:
        clean_line = line.strip()
        if clean_line != "":
            lines.append(clean_line)

    # 2. 创建字典，用来记录每个名字"已经出现过的次数"
    counts = {}

    # 3. 存放最终输出的结果
    output = []

    # 4. 遍历输入的每一行（顺序不会乱）
    for name in lines:
        if name not in counts:
            # 情况A：第一次看到这个名字
            counts[name] = 1          # 记录：出现1次了
            output.append(name)       # 原样输出
        else:
            # 情况B：以前看到过这个名字
            counts[name] += 1         # 次数加1（比如变成2）
            # 构造新名字：原名字 + （当前次数 - 1）
            # 如果是第2次，加 "1"；第3次加 "2"
            new_name = name + str(counts[name] - 1)
            output.append(new_name)

    # 5. 把结果输出，用换行符拼接
    sys.stdout.write("\n".join(output))

# 调用主函数
if __name__ == "__main__":
    main()
```


counts 字典就像一个小本本，记录每个名字你见过几次。

第一次见，原样放进输出。

第二次见，把次数从 1 变成 2，然后追加 "1"。

第三次见，变成 3，追加 "2"。

像 `sync_ams_ts2` 这种名字，因为字典里没有完全一模一样的 'sync_ams_ts2'，所以被当作第一次见，完美符合样例。


### 第五步：总结规律（再也不怕这类题）

这类“去重并编号”的题目，万能公式是：

初始化一个空字典 counts = {}。

遍历列表。

如果名字不在字典里，设值为 1，并原样输出。

如果名字在字典里，把它 +1，并输出 名字 + str(旧次数)。

注意保持顺序（用列表存储输出结果即可）。


##  REST API: Country Codes

Implement a function that formats phone numbers with the appropriate country calling codes.

Given a country name and a phone number, your function should:

1. Query the API at https://jsonmock.hackerrank.com/api/countries?name=<country>
   (replace "<country>" with the actual country name) to retrieve the country's calling codes.
2. If multiple calling codes exist, use the one at the highest index (the last one in the array).
3. Format the phone number as: "+<Calling Code> <Phone Number>"
   Example: "+93 656445445"
4. Return "-1" if the country data array is empty (country not found).

The API response contains a "data" field which is an array:

- If the country is found, the array contains exactly 1 element with country information.
- If not found, the array is empty.

The country record includes:

- name: The country name (String)
- callingCodes: An array of the country's calling codes (String Array)
- Other fields not relevant for this task

```
{
	"name": "Afghanistan",
	"callingCodes": [
	"93" 
    ],
	"capital": "Kabul"
}
```

Function Description:

Complete the getPhoneNumbers function with parameters:

- string country: the country to query
- string phoneNumber: the phone number

Returns:

- string: the completed phone number or "-1"

Constraints:

- The returned JSON object contains either 0 or 1 record in data.
- The country name may contain uppercase/lowercase letters and spaces (ASCII 32).

----


### 题目在说什么？

想象你是一个国际快递员，要给全球各地的人打电话确认收货地址。但是：

你只知道国家名字（比如 "Afghanistan"）和电话号码（比如 "656445445"）。

不同国家的国际区号（calling code）不一样，阿富汗是 +93，美国是 +1。

你需要调用一个网络接口（API），输入国家名字，它会返回这个国家的区号。

拿到区号后，你把区号和电话号码拼在一起，变成 "+93 656445445" 这样的格式。

如果这个国家不存在（查不到），就返回 "-1"。




1. 一个国家可能有多个区号（比如波多黎各有 1787 和 1939 两个），题目要求使用最高索引的那个（也就是数组的最后一个）。记住：数组的最后一个 = 索引最大的那个。

2. 拼接格式：+ + 区号 + 空格 + 电话号码。注意空格只有一个，不要多也不要少。

3. 返回 "-1" 而不是 "-I"（注意看第二个截图里写的是 "-I"，但第三个截图里明确写的是 "-1"，以最后一个截图为准）。


### 第三步：你要实现什么？

```
def getPhoneNumbers(country, phoneNumber):
    # 你的代码在这里
    # 1. 调用 API
    # 2. 解析返回的 JSON
    # 3. 取出区号（如果是多个，取最后一个）
    # 4. 拼接成 "+区号 电话号码"
    # 5. 如果查不到，返回 "-1"
```

步骤1：怎么调用 API？


```
https://jsonmock.hackerrank.com/api/countries?name=<country>
```

把 <country> 换成真实的国家名，比如：

```
https://jsonmock.hackerrank.com/api/countries?name=Afghanistan
```

步骤2：API 返回了什么？

以阿富汗为例，API 返回的 JSON 长这样：


```
{
    "data": [
        {
            "name": "Afghanistan",
            "callingCodes": ["93"],
            "capital": "Kabul"
        }
    ]
}
```

data 是一个数组。

如果找到了，数组里有一个元素（就是一个字典），里面有 callingCodes 字段。

如果没找到，data 是空数组 []。


**步骤3：怎么取区号？**

- 先判断 data 是不是空的。如果是空的，直接返回 "-1"。
- 如果不是空的，取出第一个元素（也是唯一的那个）data[0]。
- 从里面取出 callingCodes 字段。
- callingCodes 是一个数组。如果它里面有多个区号，取 最后一个（即索引 -1）。


**步骤4：怎么拼接？**

用 f-string 或者字符串拼接：

```
result = "+" + code + " " + phoneNumber
```

```
result = f"+{code} {phoneNumber}"
```


**步骤5：异常处理**

如果网络请求失败（比如超时、连接错误），也应该返回 "-1"，避免程序崩溃。

```
import requests   # 用于发送 HTTP 请求

def getPhoneNumbers(country, phoneNumber):
    # 1. 构造 API 请求的 URL
    url = "https://jsonmock.hackerrank.com/api/countries"
    params = {"name": country}   # 把国家名作为查询参数

    try:
        # 2. 发送 GET 请求到 API
        response = requests.get(url, params=params)
        # 3. 检查请求是否成功（状态码 200）
        response.raise_for_status()
        
        # 4. 把返回的内容解析成 JSON（变成 Python 字典）
        data = response.json().get("data", [])
        
        # 5. 如果 data 是空的，说明没找到这个国家
        if not data:
            return "-1"
        
        # 6. 取出第一个记录（也是唯一的一个）
        record = data[0]
        
        # 7. 取出 callingCodes 数组
        calling_codes = record.get("callingCodes", [])
        
        # 8. 如果 callingCodes 是空的（理论上不可能，但防御一下）
        if not calling_codes:
            return "-1"
        
        # 9. 取最后一个区号（最高索引）
        code = calling_codes[-1]
        
        # 10. 拼接成 "+区号 电话号码" 并返回
        return f"+{code} {phoneNumber}"
    
    except (requests.RequestException, ValueError, KeyError):
        # 11. 任何异常（网络错误、解析错误等）都返回 "-1"
        return "-1"
```


第六步：用样例测试一下

```
getPhoneNumbers("Afghanistan", "656445445")
```

API 返回 callingCodes = ["93"]

取最后一个："93"

拼接："+93 656445445" ✅

样例2：波多黎各

```
getPhoneNumbers("Puerto Rico", "564593986")
```

API 返回 callingCodes = ["1787", "1939"]

取最后一个："1939"（因为 callingCodes[-1] 是最后一个）

拼接："+1939 564593986" ✅

样例3：不存在的大洋洲

```
getPhoneNumbers("Oceania", "987574876")
```

PI 返回 data = []（空数组）

返回 "-1" ✅



1. 用 API：用 `requests.get(url, params={"name": country})`

2. 取区号：`data[0]["callingCodes"][-1]`（如果 data 不空）

2. 拼接返回：`f"+{code} {phoneNumber}”` 或 `"-1”`

```
┌─────────────────┐
│  开始：你拿到    │
│  国家名 + 电话号码 │
│  例：Afghanistan  │
│     656445445     │
└────────┬─────────┘
         │
         ▼
┌─────────────────┐
│  步骤1：打电话    │
│  给API服务器      │
│  （发请求）       │
└────────┬─────────┘
         │
         ▼
┌─────────────────┐
│  API服务器返回    │
│  JSON数据         │
│  {data: [...]}   │
└────────┬─────────┘
         │
         ▼
┌─────────────────┐
│  步骤2：检查      │
│  data是空的吗？   │
└────────┬─────────┘
         │
    ┌────┴────┐
    │  是     │  否
    ▼         ▼
┌──────┐  ┌─────────────────┐
│返回  │  │  步骤3：取出     │
│"-1"  │  │  callingCodes    │
└──────┘  │  数组           │
          └────────┬─────────┘
                   │
                   ▼
          ┌─────────────────┐
          │  步骤4：取最后   │
          │  一个区号        │
          │  (最高索引)      │
          └────────┬─────────┘
                   │
                   ▼
          ┌─────────────────┐
          │  步骤5：拼接     │
          │  "+区号 电话"   │
          └────────┬─────────┘
                   │
                   ▼
          ┌─────────────────┐
          │  返回结果        │
          │  "+93 656445445"│
          └─────────────────┘
```

```
API返回的JSON（像快递包裹）:
┌──────────────────────────────────────────┐
│ {                                        │
│   "data": [                              │
│     {                                    │
│       "name": "Afghanistan",             │
│       "callingCodes": ["93"],   ← 数组里只有1个 │
│       "capital": "Kabul"                 │
│     }                                    │
│   ]                                      │
│ }                                        │
└──────────────────────────────────────────┘
                │
                │  提取 callingCodes
                ▼
        ┌───────────────┐
        │ callingCodes  │
        │   = ["93"]    │
        └───────┬───────┘
                │  取最后一个 [-1]
                ▼
          ┌──────────┐
          │  code    │
          │  = "93"  │
          └──────────┘
                │
                │  拼接
                ▼
        ┌──────────────────┐
        │ "+93 656445445"  │
        └──────────────────┘
```

```
        输入
    ┌────────────┐
    │ 国家名 + 电话│
    └──────┬─────┘
           │
           ▼
    ┌──────────────────────┐
    │ 1. 调用API查询区号    │
    │    GET /countries?   │
    │    name=Puerto Rico  │
    └──────┬───────────────┘
           │
           ▼
    ┌──────────────────────┐
    │ 2. API返回JSON数据    │
    │    {"data": [...]}   │
    └──────┬───────────────┘
           │
           ▼
    ┌──────────────────────┐
    │ 3. data是空的吗？    │
    └──────┬───────────────┘
           │
    ┌──────┴──────┐
    │  是         │ 否
    ▼             ▼
┌─────┐    ┌──────────────────────┐
│ -1  │    │ 4. 取最后一个区号     │
└─────┘    │    callingCodes[-1]  │
           └──────┬───────────────┘
                  │
                  ▼
           ┌──────────────────────┐
           │ 5. 拼接结果          │
           │    "+区号 电话"      │
           └──────┬───────────────┘
                  │
                  ▼
           ┌──────────────────────┐
           │ 6. 返回格式化结果     │
           │    "+1939 564593986" │
           └──────────────────────┘
```


## Python: Log Decorator

Implement a decorator that logs the invocation of the decorated function to the provided file descriptor. You can assume that it will only decorate functions that take in positional arguments.

- The log line should follow this format: LOG: <function_name>(<comma_separated_call_parameters>). 
  The last character should be a newline.
- The line should be logged prior to the execution of the decorated function's body.

Example:

```
@log(descriptor)
def my_max(a, b, c):
    return max(a, b, c)

@log(descriptor)
def my_min(a, b):
    return min(a, b)

my_max(20, 9, 10)
my_min(1, 2)
```

Writes to the given descriptor the following output:

- LOG: `my_max(20, 9, 10)`
- LOG: `my_min(1, 2)`

Your implementation of the decorator will be tested by a provided code stub on several input files. Each input file contains parameters to call a decorated function. The provided code calls the functions and writes the returned values to the descriptor.

Constraints:

The decorated function takes only positional arguments.

Input Format for Custom Testing:

Sample Input 0:

```
STDIN   3
my_max 2 7 1
my_min 7 1
my_sum 2 7 1 10
```

Sample Output 0:

```

LOG: my_max(2, 7, 1)
7
LOG: my_min(7, 1)
1
LOG: my_sum(2, 7, 1, 10)
20
```

Explanation 0

There are 3 function calls to be executed:

- The first one calls the decorated my_max function to return a maximum value of 2, 7, 1. 
  The decorator prints LOG: my_max(2,7,1) to the given descriptor and the provided code writes the returned value, 7.
- The second one calls the decorated my_min function to return a minimum value of 7, 1. 
  The decorator prints LOG: my_min(7,1) and the code writes the returned value, 1.
- The third one calls the decorated my_sum function to return the sum of 2, 7, 1, 10. 
  The decorator prints LOG: my_sum(2,7,1,10) and the code writes the returned value, 20.


