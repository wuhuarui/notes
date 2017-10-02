## sort algorithms

### merge sort

method in python:

```python
import operator

def mergeSort(L, compare = operator.lt):
    """
    mergeSort function
    Given an arbitrary list L, and compare operator
    Return the sorted list.
    The complexity is O(n*log(n)), n is the length of list
    """
    if len(L)<2:
        return L[:]
    else:
        middle = int(len(L)/2)
        left = mergeSort(L[:middle], compare)
        right = mergeSort(L[middle:], compare)
        return merge(left, right, compare)



def merge(left, right, compare):
    """
    merge two ordered list: left and right in the method of compare-ordered

    Given left, right: two lists and a compare operator
    Returns an ordered list contains the elements in left and right
    """
    global nCalls
    nCalls += 1
    print('Called merge function for', nCalls, 'times.')
    i, j = 0, 0
    result = []
    while i < len(left) and j < len(right):
        if left[i] < right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1
    if i < len(left):
        result = result + left[i:]
    else:
        result = result + right[j:]
    return result


global nCalls
nCalls = 0
mergeSort([5,7,3,6,9,2,10,0,4])
# print(merge([1,5,7,9,10],[3,4,5,7,8,9],operator.lt))
```

Complexity of merge sort is n*log(n), n is the length of input list.

Why n*log(n):

Actually the complexity should be the combination of `mergeSort` complexity and `merge` complexity.

The complexity of `merge` is O(len(L)). So the complexity is O(len(L))* number of `merge` method called.

For a list with length of n, the number of `merge` be called is: 1 + 2 + 4 + ... + n/2 ($log_2n$ elements in total)

However, the merge length is different, for the first `merge`, the length is n, for the second two `merge`, the length is n/2.

In this way, the complexity is: 1*n + 2*(n/2) + ... + (n/2)*2 = n*log(n).
