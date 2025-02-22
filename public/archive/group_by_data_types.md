---
title: "Group by data types"
category: ''
pubDate: 'Apr 02 2024 16:37'
---

I came across an interesting problem today. Given an array with various values, the task is to extract the data based on their types, regroup them, and return those groups in an array. You can assume that the given array will never be empty. So here's an example. Let say the input is `[1, true, 2, false]`, the expected output should be `[[1, 2], [true, false]]`.

Here's another example. If the input is `[[1, 2], 1, null, true]`, the output should be `[[[1,2]], [1], [null], [true]]`. In this case, there are four types present in the array: _objects_, _numbers_, _null_ values, and _booleans_.

```js
function sortTypes(arr) {  
  let answer = []
  return answer
}
 ```

Below is my solution to the problem.

```js
function sortTypes(arr) {
	let answer = []
	let map = new Map()

	arr.forEach(data => {
		if(data === null) {
			if(map.has('null')) {
				map.get('null').push(data)
			} else {
				map.set('null', null)
			}
		}
		else if(map.has(typeof data)) {
			map.get(typeof data).push(data)
		} else {
			map.set(typeof data, [data])
		}
	})

	for(let [key, value] of map) {
		answer.push(value)
	}

	return answer
}

let x = sortTypes([1, true, 2, false])
let y = sortTypes([[1,2],1,null,true, undefined])
console.log(x)
console.log(y)
```

The operations used with `map` (`map.get`, `map.has`, `map.set`) are generally O(1). I used two loops using `forEach` and `for..of` which is linear.

The overall time complexity of this algorithm is O(N). 