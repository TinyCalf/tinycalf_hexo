---
title: javascript去重算法研究
date: 2019-01-28 11:34:12
tags:
---


```javascript
let array = [1, 1 , "1"]

function deduplication(arr) {
	let res = []
	arr.forEach(item => {
		if(res.indexOf(item)=== -1) {
			res.push(item)
		}
	})
	return res
}

console.log(deduplication(array))
```


```javascript

```
