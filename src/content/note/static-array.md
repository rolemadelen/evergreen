---
title: "Static Array"
category: "ea1a"
pubDate: "Dec 30 2024 18:00:00"
---

There are [different types of an array](/note/different-types-of-an-array) and static array is one of them.

Static array is created at compile time. That's why it is _static_. Since the size of the array is fixed, you must know the size of it beforehand. Also it cannot be extended or re-scaled during the runtime.

```c
#include <stdio.h>

int main(int argc, char**argv) {
	int arr[5];

	arr[0] = 1;
	arr[1] = 2;
	arr[2] = 3;
	arr[3] = 4;
	arr[4] = 5;

	printf("%d\n", arr[0]);

	arr = arr[10]; // Compile error: cannot re-assign it

	return 0;
}

```

If you don't know the size beforehand or you need to be able to rescale the array, you should use a [[Dynamic Array]].