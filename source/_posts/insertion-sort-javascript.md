title: Insertion Sort Javascript
date: 2016-04-12 19:52:04
tags: 
      - javascript
      - algorithms
---

Insertion sort works by separating an array into two sections:

* Sorted section
* Unsorted section

Let's assume initially the array is unsorted. The sorted section is empty. The algorithm for insertion sort works like this.

1. First grab a value from the unsorted section and added to the sorted section. An array of one is always sorted.
1. Then, for each item of the unsorted array we will do:
  1. If the item value is _greater_ than our last item in the _sorted_ section, we leave it there.
  1. If the item is _less_ than our last item from the __sorted__, we take that item from the array and move the last sorted item into the new open spot that we have.
1. Then we compare the item value to the previous value in the sorted section.
  1. If the item value goes after the previous value and before the last value, then place the item into the open spot between them.
  1. If the item value is greater, then continue the process until the start of the array is reached.

```
function insertionSort(items) {
  var len   = items.length;
  var value;
  var unsortedIndex;
  var sortedIndex;

  for(unsortedIndex = 0; unsortedIndex < len; unsortedIndex += 1) {
    value = items[unsortedIndex];
    sortedIndex = unsortedIndex - 1;

    // while(sortedIndex >= 0 && items[sortedIndex] > value) {
    //   items[sortedIndex + 1] = items[sortedIndex];
    //   sortedIndex = sortedIndex - 1;
    // }

    // items[sortedIndex + 1] = value;

    for(sortedIndex = unsortedIndex - 1; sortedIndex >= 0 && items[sortedIndex] > value; sortedIndex -= 1) {
      items[sortedIndex + 1] = items[sortedIndex];
    }

    items[sortedIndex + 1] = value;
  }

  return items;
}

console.log(insertionSort([5, 2, 6, 1, 3, 9]));
```


Honestly when I first saw this, it was so counterintuitive for me, so what I do to help me to vizualise this kind of problems is to manually be the compiler, in the case of javascript, the manual interpreter.


In this example, I will use the `while` loop rather than `for`, because it's a little bit more verbose:

{% youtube 3ueQcylS4m4 %}