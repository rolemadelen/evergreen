---
title: "Quickest way to pad zeroes"
category: 'JS'
pubDate: 'May 18 2024 14:53'
---

Let's say you want to ensure that numbers are always represented in two digits, padding zeroes if the number is a single digit.

One way is to create a function to check if a number is less than 10 and return a string with padded zeroes.

```js
const padZero = (num) => {
	if(num < 10) return `0${num}`
	else return `${num}`
}

console.log(padZero(3))
```

There's a simpler way to achieve this using JavaScript's internal method in a string called `padStart`.

```js
const num = 3
console.log(num.toString().padStart(2, '0')) // 03
console.log(num.toString().padStart(3, '0')) // 003
console.log(num.toString().padStart(4, '0')) // 0003
```

Here's an example. Let's say you're working with time. We can use the above method to pad zeroes for hours, minutes, and seconds when they are less than 2 digits.

```js
const pad = (time, count) => {
	return time.toString().padStart(count, '0');
}

const Time = ({hour, minute, second}) => ({
	hour,
	minute,
	second,
	print() {
		return `${pad(hour, 2)}:${pad(minute, 2)}:${pad(second, 2)}`
	}
})

const date = new Date()
const currentTime = Time({
	hour: date.getHours(),
	minute: date.getMinutes(),
	second: date.getSeconds()
})

console.log(currentTime.print())

```


## References
- Dasore, H. (2022, January 5). Simplest way to add a leading zero to a number in Javascript. Medium. https://hardiquedasore.medium.com/simplest-way-to-add-a-leading-zero-to-a-number-in-javascript-b8724749486f