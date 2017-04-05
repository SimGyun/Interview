# 面试小记
纯粹记下面试中我觉得有意思的问题

## add(2)(5)的JavaScript累加

上周五去面试，最后面试官说做道题吧。就直接扔了张纸过来，上面就写了两行
```
add(2,5)           //7
add(2)(5)          //7
```
我一看，这不简单嘛！于是洋洋洒洒写下了
```
    const add = (a)=> (b)=> a + b;
```
写完还想怎么这么简单。果然，面试完回来上网查了一下，原来面试官想要的答案是随意参数累加啊！
实现这个功能，我想了几个关键的地方:
- 对于不确定个数的参数，可以用ES6的rest参数，详情看[阮老师的教程](http://es6.ruanyifeng.com/#docs/function#rest参数)，当然也可以用es5的arguments类数组对象
- 实现累加的方法 **Array.prototype.reduce()**,或者forEach()也行.
- 因为实现累加的函数调用**accu()**返回的还是函数，所以需要重写accu的**valueOf**方法，以返回需要的累加值。
说到valueOf,就简单说一下。
在下面两种情况下，函数会自己调用valueOf()或者toString()
- 函数的返回值不是基本数据类型(Number,String,Boolean,null,undefined，Symbol),说到symbol,有个小疑问，文末提.
- 函数的 valueOf() 和 toString() 任一或两者被override
在 valueOf() 和 toString() 都被override的情况下，会优先调用valueOf(),如果valueOf()返回的不是基本数据类型，再调用toString()
于是，答案应该是下面这样。
```
    const add = (...args)=> {
        let sum = args.reduce((pre,crt)=> pre + crt);
        const accu = (...args2)=> {
            sum += args2.reduce((pre,crt)=> pre+crt);
            return accu;
        }
        accu.valueOf = ()=> sum;
        return accu;
    }
```
##### 小疑惑
- 返回值是**Symbol**
- valueOf()以及toString()的返回值都不是基本数据类型 
在以上两种情况下 我把返回值打印出来 但是在控制台是这样的
```
    #<Function>
```
测试代码如下:
```
const testA = ()=> {
    const fn =()=> {
      console.log('called');
      return fn;
    }
    fn.valueOf = ()=> {
      return [];
    }
    fn.toString = ()=> {
      return {};
    }
    return fn;
  } 
  console.log(testA()());
```
嗯，这个打印应该是浏览器的底层实现吧?!
待我深究后，再更新。

## 字符串匹配------exec()与match()

原题是这样的
在 'abcdefg' 中匹配 'efg'.
一看是不是很简单? 但是当时我在用exec()还是match()之间纠结了一会。。

##### 当正则是非全局匹配模式时，
此时，RegExp的exec()返回的结果与String.match()返回的结果是相同的。
```
  let reg = /efg/;
  let str = 'abcdefgefgefg';

  const matchStr = (str)=> str.match(reg);

  const execStr = (str)=> reg.exec(str);

  matchStr(str);   //["efg", index: 4, input: "abcdefgefgefg"]
  execStr(str);    //["efg", index: 4, input: "abcdefgefgefg"]

```
此时，match()和exec()都只执行一次匹配.若没有找到任何匹配的文本，返回null。否则，返回一个数组。
其中，第0个元素存放的是匹配文本，index 属性声明的是匹配文本的起始字符在 string 中的位置，input 属性声明的是对 string 的引用。

##### 当正则是全局匹配模式时
1. match()返回一个数组，里面存放所有的匹配子串,不会声明匹配子串的位置，所以没有input和index属性。

```
  let reg = /efg/g;
  let str = 'abcdefgefgefg';
  const matchStr = (str)=> str.match(reg);  //["efg", "efg", "efg"]
```

2. exec()在RegExp对象的lastIndex属性指定的字符处开始检索字符串,默认等于0。当 exec() 找到了与表达式相匹配的文本时，在匹配后，把 RegExp 的 lastIndex 属性设置为匹配文本的最后一个字符的下一个位置。直到匹配完成，返回结果null.因此，在循环中反复地调用 exec() 方法可获得全局模式的完整检索信息。
```
  let reg = /efg/g;
  let str = 'abcdefgefgefg';
  const execStr = (str)=> {
      let result;
      while(result = reg.exec(str)){
        console.log(result);
      }
  }
  execStr(str);  // ["efg", index: 4, input: "abcdefgefgefg"]
                 // ["efg", index: 7, input: "abcdefgefgefg"]
                 // ["efg", index: 10, input: "abcdefgefgefg"]
```

因此，用 match() 还是 exec(), 就看你需要什么信息了。

## 出现次数最多的字符
这次还写了一道 “求一个字符串中出现次数最多的字符，以及出现多少次”。当时脑海中马上闪出 Object.entries()。
但这个方法是es7提出的草案，目前处于试验中的功能。
不过既然是捣腾代码，就试一下吧~

1. 首先可以用一个Object记录相应的字符出现了多少次，获取每个字符的方法 **stringObject.charAt()**
2. 遍历对象，找出次数最多的字符以及次数.这里我用的是 **Object.entries()**;

```
  const getChar = (str)=> {
      let obj = {};
      let len = str.length;
      for(let i = 0;i<len;i++){
        let char = str.charAt(i);
        if(!obj[char]){
          obj[char] = 1;
        }else{
          obj[char]++;
        }
      }
      let count = 0 , char = '';

      // 用es5的for in 循环，或者Object.keys()都可以

      for(let [key,value] of Object.entries(obj)){
        if(value > count){
          char = key;
          count = value;
        }
      }
      console.log(`character ${char} had shown up ${count} times`);
   }
   getChar('sasdgthgsfsggaasssss');   // character s had shown up 9 times
```
