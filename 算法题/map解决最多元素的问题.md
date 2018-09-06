现有一个数组，如何得出数组中的元素的值出现最多的那一个元素。

> 比如[1,1,2,3,3,5,4,4,4] 中出现最多的是4

代码如下：

```js
/* map的结构如下
{
    val: arr[i],
    num: 0, // 表示arr[i]出现的次数
    index: '1', // 表示val出现在arr的哪些位置
} */

function maxTime(arr){
    const m = new Map()
    let tmp
    for(let i = 0; i < arr.length; i++){
        if(m.has(arr[i])){
            tmp = m.get(arr[i])
            tmp.num++
            tmp.index += `,${i}`
            m.set(arr[i], tmp)
        } else {
            m.set(arr[i], {
                val: arr[i],
                num: 1,
                index: `${i}`
            })
        }
    }
    let max = 0, result
    m.forEach(function(value){
        if(max < value.num){
            max = value.num
            result = value
        }
    })
    return result
}

let arr = [1,2,3,1,2,3,4,4,4,4,46,5,6,7,]
console.log(maxTime(arr)) // { val: 4, num: 4, index: '6,7,8,9' }
```

