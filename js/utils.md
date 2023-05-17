记录一些实用的封装函数

# 防抖函数

```js
// 防抖函数

export function debounce(fn, delay) {

  let timer

  return function () {

​    if (timer) {

​      clearTimeout(timer)

​    }

​    timer = setTimeout(() => {

​      fn()

​    }, delay)

  }

}
```

# 节流函数

```js
var click_t = null
// 节流函数

export function throttle(fun, delay = 2000) {

  if (!click_t) {

​    fun()

​    click_t = setTimeout(() => {

​      clearTimeout(click_t)

​      click_t = null

​    }, delay)

  }

}
```

# 文本域特殊字符转为html

```js
const textareaToHtml = (str) => {
	if (str == null) {
		return "";
	} else if (str.length == 0) {
		return "";
	}
	str = str.replace(/\r\n/g, "<br>")
	str = str.replace(/\n/g, "<br>");
	str = str.replace(/\r/g, "<br>");
	str = str.replace(/( )/g, "&nbsp;")
	return str;
}

```

# 获取日期

```js
//day 为天数，0为当天5为5天后，-5为5天前
export function get_date(day) {
    let myDate = new Date();
    myDate.setTime(myDate.getTime() + day * (24 * 60 * 60 * 1000));
    let y = myDate.getFullYear();    //获取完整的年份(4位,1970-????)
    let m = myDate.getMonth() + 1;       //获取当前月份(0-11,0代表1月)
    let d = myDate.getDate();        //获取当前日(1-31)
    let hh = myDate.getHours();
    let mm = myDate.getMinutes();
    let ss = myDate.getSeconds();
    if (m < 10) {
        m = "0" + m
    }
    if (d < 10) {
        d = "0" + d
    }
    if (hh < 10) {
        hh = "0" + hh
    }
    if (mm < 10) {
        mm = "0" + mm
    }
    if (ss < 10) {
        ss = "0" + ss
    }
    return y + "-" + m + "-" + d + " " + hh + ":" + mm + ":" + ss
}
```

