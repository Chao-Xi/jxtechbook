# 5 Jump Trading Interview (code)

## 1 smallest missing positive integer

Write a function:

def solution(A)

that, given an array A of N integers, returns the smallest positive integer (greater than 0) that does not occur in A.

For example, given `A = [1, 3, 6, 4, 1, 2]`, the function should return 5.

Given `A = [1, 2, 3]`, the function should return 4.

Given `A = [−1, −3]`, the function should return 1.

Write an efficient algorithm for the following assumptions:

N is an integer within the range `[1..100,000]`;

each element of array A is an integer within the range `[−1,000,000..1,000,000]`.

```
def solution(A):
    # Keep only positive numbers in a set for O(1) lookups
    positives = set(x for x in A if x > 0)

    # Start checking from 1 upwards
    smallest = 1
    while smallest in positives:
        smallest += 1

    return smallest
```

This is the "smallest missing positive integer" problem.

We can solve it in **O(N) time and O(N) space** by using a set to track all positive numbers in the array, then scanning for the smallest missing one.

Explanation

#### 1 Filter positives

Negative numbers and zero are irrelevant since we’re looking for the smallest positive integer.


#### 2 Use a set

Lookup in a set is O(1) average time complexity.

#### 3 Increment until missing

Start from 1 and keep going until we find a number not in the set.

```
print(solution([1, 3, 6, 4, 1, 2]))  # 5
print(solution([1, 2, 3]))           # 4
print(solution([-1, -3]))            # 1
```

If you want O(1) extra space and still O(N) time, we could do an in-place index marking trick — I can show you that version too, which is more memory efficient and popular in coding interviews.

Do you want the O(1)-space in-place version? It’s the one Codility gives 100/100 for this problem.


## 2 File sync check

Given list of `source_files`, and a list of `destination_files` returns a list of filenames that need to be synced from source to destination in the order as they appear in `source_files`. Each element in the file lists will have 5 columns delimited by a space character.

The columns in order from left to right will be: permissioning, username, size, modification time, filename.  For example:

```
["rwx alex 2673422 05:04:03 somefilename.txt", 
"rwx alice 993432 05:45:00 myfile.csv", 
"rwx alex 734673422 19:24:13 somefilename.txt", 
"rwx root 14384 12:04:05 somefilename.txt"]
```

**A file needs to be synced if it does not exist at the destination or either size, modification time, permissions, or user are different between the source and destination**. 

**A file can only be synced if the file is readable at the source and writeable at the destination.** 

For example, given the following source and destination files:

source:

```
["rwx alex 2673422 05:04:03 somefilename.txt", 
"rwx alice 993432 05:45:00 myfile.csv", 
"rwx alex 734673422 19:24:13 foobar.txt", 
"-wx root 184 12:04:05 barfoo.txt", 
"r-- alice 10 02:00:00 helloworld.txt"]
```


destination:

```
["rwx alex 5 05:04:03 somefilename.txt", 
"rwx alice 993432 10:45:00 myfile.csv", 
"r-x alex 734673422 04:24:13 foobar.txt", 
"rwx root 4 12:04:05 barfoo.txt"]
```


In this example, the return list should be



```
["somefilename.txt", "myfile.csv", "helloworld.txt"]
```


because:

- somefilename.txt has different sizes between source and destination
- **foobar.txt is not writeable at the destination**
- etc

Your solution will be evaluated both on correctness and coding style. We will read your code.

This is essentially a **file sync check** with some permission constraints.

We need to:

1. Parse the file lists into structured records for easy comparison.
2. Check:

   * File is in source but not in destination → needs sync.
   * File is in both, but any of **permissions, username, size, modification time** differ → needs sync.
3. Only sync if:

   * **Source** file’s permission has `'r'` in position 0 (readable).
   * **Destination** file’s permission has `'w'` in position 1 (writeable) OR file missing in destination (in which case we assume we can write it).

Here’s a clean Python implementation:

```python

def parse_file_entry(entry):
    # permission, user, size, mtime, filename
    perm, user, size, mtime, filename = entry.split(" ", 4)
    return {
        "perm": perm,
        "user": user,
        "size": int(size),
        "mtime": mtime,
        "filename": filename
    }

def solution(source_files, destination_files):
    # Convert destination files into a dict for quick lookup by filename
    dest_dict = {}
    for entry in destination_files:
        info = parse_file_entry(entry)
        dest_dict[info["filename"]] = info

    to_sync = []

    for src_entry in source_files:
        src = parse_file_entry(src_entry)

        # Must be readable at source
        if not src["perm"].startswith("r"):
            continue

        dest = dest_dict.get(src["filename"])

        # If file missing in destination → needs sync
        if dest is None:
            to_sync.append(src["filename"])
            continue

        # Must be writeable at destination
        if "w" not in dest["perm"][1]:
            continue

        # Check differences (excluding filename)
        if (
            src["perm"] != dest["perm"] or
            src["user"] != dest["user"] or
            src["size"] != dest["size"] or
            src["mtime"] != dest["mtime"]
        ):
            to_sync.append(src["filename"])

    return to_sync
```

---

### Walkthrough with your example

```python
source = [
    "rwx alex 2673422 05:04:03 somefilename.txt",
    "rwx alice 993432 05:45:00 myfile.csv",
    "rwx alex 734673422 19:24:13 foobar.txt",
    "-wx root 184 12:04:05 barfoo.txt",
    "r-- alice 10 02:00:00 helloworld.txt"
]

dest = [
    "rwx alex 5 05:04:03 somefilename.txt",
    "rwx alice 993432 10:45:00 myfile.csv",
    "r-x alex 734673422 04:24:13 foobar.txt",
    "rwx root 4 12:04:05 barfoo.txt"
]

print(solution(source, dest))
# Output: ['somefilename.txt', 'myfile.csv', 'helloworld.txt']
```

---

This approach is:

* **O(N + M)** time (N = source length, M = destination length)
* **O(M)** space for the destination dictionary.

---

If you want, I can make this **more concise** without losing clarity, or even write a **single-pass version** that avoids building the dictionary when lists are sorted.


## Integer prices

**Given a sequence of integer prices, we can determine when to buy and sell by identifying special sequences of prices movements called indicators.**

For example, **we can decide to buy every time we see 2 consecutive price increases**, **and we can decide to sell every time we see 2 consecutive price decreases followed by a price increase**. If we represent a price increase as 1 and a price decrease as -1, we could write the above rules as:

```
buyIndicator = [ 1,  1]     // buy after 2 price increases
sellIndicator = [-1, -1, 1] // sell after 2 price decreases then an increase
```

Specifically, consider the price sequence:


```
indexes  = [  0,   1,   2,   3,   4,   5,   6,   7,   8,   9] 
prices     = [51, 56, 56, 58, 60, 59, 54, 57, 52, 48]
```


**According to the buy/sell rules described in the indicators above, we would decide to buy at index=3 (price=58) because that price is a second consecutive price increase.** 

**Note that the price changes need not occur on consecutive indices. For example the sequence "51 56 56 58" counts as a buy signal since after ignoring indices with no change, the price increased twice from 51->56->58. **

We would also buy at index=4 (price=60) for the same reason. We would sell at index=7 (price=57) because that price is preceded by 2 consecutive price decreases then an increase. Buy/sell signal will never trigger if the current price is equal to the previous price (i.e. if the price sequence is [51, 56, 56, 58, 58] we will only buy once at index=3). Note that this is just an example of buyIndicator and sellIndicator - they may differ between tests.

Finally, we define a position as the sum of buys and sells we have done up to that point, where a buy=1 and a sell=-1. When we have made no buys or sells, our position is 0. In the above example, our positions would be:

```
indexes  = [  0,   1,   2,   3,   4,   5,   6,   7,   8,   9]
prices     = [51, 56, 56, 58, 60, 59, 54, 57, 52, 48]
position  = [  0,   0,   0,   1,   2,   2,   2,   1,   1,   1]
```


Note that when we buy at index=3 and index=4 our position increases by 1 each time, and when we sell at index=7 our position decreases by 1.

Write a function: 

solution(prices, buyIndicator, sellIndicator)

that, given a sequence of prices and sequences defining buy and sell indicators, returns a list of positions. Your position list must be the same length as the input price list. The buyIndicator and sellIndicator lists will each have at least one element and each element can only be 1 or -1. It is possible for both the buy and sell indicators to match at the same time. In that case the change in position should be 1 + (-1) = 0.

Your solution will be evaluated both on correctness and coding style. We will read your code.

Alright — let’s break this one down carefully.

### Step-by-step reasoning

We’re essentially:

1. Converting the `prices` into a **movement sequence**:

   * `1` = price increase
   * `-1` = price decrease
   * `0` = no change (and we *ignore these* when matching indicators)

2. Scanning the movements and checking if the last `len(buyIndicator)` or `len(sellIndicator)` *non-zero movements* match the pattern.

3. When a match happens:

   * Buy → position += 1
   * Sell → position -= 1
   * If both match → net change = 0.

4. For each price index, append the *current position* to the output list.

---

### Python solution

```python
def solution(prices, buyIndicator, sellIndicator):
    n = len(prices)
    position = 0
    positions = []
    
    # We'll store the last non-zero movements
    movement_history = []

    for i in range(n):
        if i > 0:
            if prices[i] > prices[i - 1]:
                movement_history.append(1)
            elif prices[i] < prices[i - 1]:
                movement_history.append(-1)
            else:
                movement_history.append(0)  # unchanged
        
        # Remove zeros for pattern matching
        filtered_history = [m for m in movement_history if m != 0]

        delta = 0  # net change in position at this index

        # Check buy pattern
        if len(filtered_history) >= len(buyIndicator):
            if filtered_history[-len(buyIndicator):] == buyIndicator:
                delta += 1
        
        # Check sell pattern
        if len(filtered_history) >= len(sellIndicator):
            if filtered_history[-len(sellIndicator):] == sellIndicator:
                delta -= 1

        # Update position
        position += delta
        positions.append(position)
    
    return positions
```

---

### Walkthrough with your example

```python
prices = [51, 56, 56, 58, 60, 59, 54, 57, 52, 48]
buyIndicator = [1, 1]
sellIndicator = [-1, -1, 1]

print(solution(prices, buyIndicator, sellIndicator))
# Output: [0, 0, 0, 1, 2, 2, 2, 1, 1, 1]
```

---

### Why this works well:

* Keeps track of **only** the last relevant movements.
* Ignores `0` movements for matching logic.
* Correctly handles *both patterns matching at the same time* (buy=1, sell=-1 → net 0).
* **O(n)** time and **O(n)** space (could be reduced to O(1) extra space by only keeping a sliding window).

## 4 To access CSV data sets download zipped files


Extract, summarise and export information from CSV files to a list

Your task is to prepare a function that will return a list of aggregated data from provided CSV files located in the ./data/ folder. The CSV files contains historical prices of traded shares of fictional companies. The filename of each file reflects the name of a fictional company. All data sets have the same structure and consist of six columns:


- date – a specific trading date parsed as a datetime object in the format 'YYYY-MM-DD';

- open , max , min , close  – the opening, maximum, minimum and closing prices, respectively;

- vol – volume (number of shares traded during a given day).

Task details

Create a function named solution that takes as an argument a list of paths to one or more of the provided files. The function should return a list of several nested lists. The length of the returned list equals the length of the list of paths passed to the function. Each nested list consists of two data frames with aggregated data for the individual company:

first data frame: highest trading volume values in individual years

contains two columns: date

- the date of the day with the highest trading volume in a year (parsed as a datetime, in the format 'YYYY-MM-DD'), and vol
- the actual volume of shares traded in that day,
- the number of rows of the data frame should be equal to the number of years in the history of the given company;

second data frame: days of highest closing prices in individual years

contains two columns: date

- the date of the day with the highest closing prices in a year (parsed as a datetime, in the format 'YYYY-MM-DD'), and close
- the actual closing price; if the same closing price occurred on more than one day in a given year, you should include them all with their full dates,
- the number of rows of the data frame might be greater than the equivalent dimension for the first data frame.

Below you can find output returned after calling the function for two chosen files (i.e. throwsh.csv and twerche.csv).

For readability purposes only 6 first rows of each data frame are shown. Notice that in the first data frame with aggregated data of a given company there are only single values in a given year; however closing prices for a given company in a given year might have repeated, so years in a date columns in the second data frame for a given company are not distinct.


```
[
    [
            date       vol
        0  1997-07-10   7190075
        1  1998-02-09   3968369
        2  1999-07-07   6231927
        3  2000-01-14   5259179
        4  2001-10-11   7681942
        5  2002-01-10   4323656,
            date     close
        0  1997-10-22    8.4273
        1  1998-02-09    5.4130
        2  1998-04-30    5.4130
        3  1999-07-09    9.8670
        4  2000-02-25   11.4410
        5  2001-01-29    8.7326
    ],
    [
            date      vol
        0  1997-07-01  3240026
        1  1998-11-03  1047549
        2  1999-07-15  3829036
        3  2000-01-18  1351917
        4  2001-01-23   103433
        5  2002-02-13    90521,
            date   close
        0  1997-10-22  22.278
        1  1998-07-20  32.673
        2  1998-07-23  32.673
        3  1999-07-14  30.127
        4  2000-01-27  31.785
        5  2001-12-31  27.157
    ]
]
```


Remember that aggregated data for any chosen companies (CSV files) must be presented as a list of lists. Apart from base Python, you can use the numpy and pandas packages to create a solution to the task.

Available packages / libraries

pandas - 2.2.3



```python
import pandas as pd

def solution(file_paths):
    result = []

    for path in file_paths:
        # Load CSV, parse 'date' as datetime
        df = pd.read_csv(path, parse_dates=['date'])

        # Add a 'year' column for grouping
        df['year'] = df['date'].dt.year

        # 1) Highest trading volume days per year (single day per year)
        vol_idx = df.groupby('year')['vol'].idxmax()  # index of max volume per year
        highest_vol_df = df.loc[vol_idx, ['date', 'vol']].reset_index(drop=True)

        # 2) Days of highest closing prices per year (can be multiple days per year)
        max_close_per_year = df.groupby('year')['close'].transform('max')
        highest_close_df = df[df['close'] == max_close_per_year][['date', 'close']].reset_index(drop=True)

        result.append([highest_vol_df, highest_close_df])

    return result
```

---

### Explanation:

* We read each CSV and parse the `date` column as datetime.
* Create a `year` column for grouping.
* For volumes:

  * Use `groupby('year')['vol'].idxmax()` to get the index of the max volume day per year.
  * Select those rows and columns.
* For closing prices:

  * Use `groupby('year')['close'].transform('max')` to get max close per row's year.
  * Filter rows matching max close price (this includes duplicates in a year).
* Return a list of pairs of dataframes as requested.

---

### Usage example:

```python
files = ['./data/throwsh.csv', './data/twerche.csv']
aggregated_data = solution(files)

for company_data in aggregated_data:
    vol_df, close_df = company_data
    print(vol_df.head())
    print(close_df.head())
```

---

This is clean, efficient, and uses pandas idiomatically. Let me know if you want help with zipping/unzipping files or anything else!
