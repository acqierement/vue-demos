# readme-demo2

这个示例是官方的一个示例，html界面也很简单，一个输入框和下面的表格显示。

```html
<div id='demo'>
    <form id='search'>
        Search <input name='query' v-model='searchQuery'>
    </form>
    <demo-grid 
               :data='gridData' 
               :columns='gridColumns' 
               :filter-key='searchQuery'>
    </demo-grid>
</div>
```

这个的js代码如下

```js
var demo = new Vue({
    el: '#demo',
    data: {
        searchQuery: '',
        gridColumns: ['name', 'power'],
        gridData: [
            { name: 'Chuck Norris', power: Infinity },
            { name: 'bruce Lee', power: 9000 },
            { name: 'Jackie chan', power: 7000 },
            { name: 'Jet Li', power: 8000 }
        ]
    }
})
```

还是熟悉的配方，输入框input里面用v-model与一个数据（searchQuery）进行绑定。然后在自定义控件demo-grid里面有`:data`等属性，看到冒号开头的我们知道是v-bind的缩写，所以这三列代表数据绑定，就是将父组件`#demo`的数据传入子组件，也就是传入到我们的自定义组件demo-grid里面。

所以所有的逻辑实现都在组件里面，来看看vue的组件是什么东西。

先大体来看看我们用到的demo-grid组件，这里我省略了一些细节

```js
Vue.component('demo-grid', {
    template: '#grid-template',
    props: {
        data: Array,
        columns: Array,
        filterKey: String
    },
    data: function () {
        ......
        return {
            sortKey: '',
            sortOrders: sortOrders
        }
    },
    computed: {
        filteredData() {
            ......
            return data;
        }
    },
    filters: {
        capitalize(str) {}
    },
    methods: {
        sortBy(key) {}
    }
})
```

- **template**：html模板，这次例子的模板是放在script里，主要实现了数据显示，具体可以看代码，大体就是for循环显示数据，增加click属性和class属性。这里在标题栏，就是name和power上面的显示用了filters，就是下面马上写到的。
- **props**：我们发现props里面定义的就是前面组件里面绑定的属性，就是在props里面定义的，包括定义它们的数据类型。
- **data**：组件里面的data必须是函数，这里返回了`sortKey`和`sortOrders`，sortKey表示是用name排序，还是用power排序，sortOrders表示当前name和power的排序顺序（顺序or逆序）。
- **computed**：额外需要计算的数据。有些传入的数据不是我们直接需要的数据，比如传入数字a和b，我们要展示的是和，则可以在computed里面求和返回。我们这里例子里面，主要是根据搜索词进行筛选，还有根据排序顺序，将数据排序一下，最后返回处理好的数据，注意这里的data是函数自己的定义的变量，和外面的data没有关系，其实如果要使用外面的data，应该使用this.data。
- **filters**：自定义过滤器。这个例子里的过滤函数是`capitalize(str)`，作用就是把传入的参数首字符变成大写。来看一下我们怎么使用的。

```html
<th v-for="key in columns">
    <!--capitalize 是自定义过滤器函数，key 为capitalize的第一个参数-->
    {{ key | capitalize }} 
</th>
```

在两个花括号里面使用过滤器，columns里面就两个数据，name和power，最后显示数据就是Name和Power。

- **methods**：定义了点击的函数，实现排序，当真正的排序逻辑是在computed里面实现的，这里只是更改了前面data里面提到的`sortKey`和`sortOrders`。根据这两个数据来决定要展示什么数据。

前面说了根据搜索词筛选和排序都是在computed里面实现的，我们来好好看看这里面函数的代码。

```js

computed: {
    filteredData() {
        var sortKey = this.sortKey;
        var filterKey = this.filterKey && this.filterKey.toLowerCase();
        var order = this.sortOrders[sortKey] || 1;
        var data = this.data;
        //根据搜索词进行匹配，展示匹配的数据
        if (filterKey) {
            data = data.filter(function (row) {
                return Object.keys(row).some(function (key) {
                    return String(row[key]).toLowerCase().indexOf(filterKey) > -1;
                })
            })
        }
        //判断要个根据name排序还是根据power进行排序
        if (sortKey) {
            data = data.sort(function (a, b) {
                a = a[sortKey];
                b = b[sortKey];
                return (a === b ? 0 : a > b ? 1 : -1) * order
            })
        }
        return data;
    }
},
```

因为我原本是学后端的，对前端不是很了解，这里面的js函数好多都不懂，现在我从上到下来讲解一下这些js函数。

- **filter**：它用于把Array的某些元素过滤掉，然后返回剩下的元素。Array的filter()接收一个函数，filter()把传入的函数依次作用于每个元素，然后根据返回值是true还是false决定保留还是丢弃该元素。现在传入的参数是每一行(row)。
- **Object.keys(row)**：key的数组，这里我们的key就是name和power，比如下面这个例子

```js
 var obj = { 0: 'a', 1: 'b', 2: 'c' };
 console.log(Object.keys(obj)); // console: ['0', '1', '2']
```

- **some()**： some() 方法用于检测数组中的元素是否满足指定条件（函数提供）。 some() 方法会依次执行数组的每个元素，如果有一个元素满足条件，则表达式返回true , 剩余的元素不会再执行检测。如果没有满足条件的元素，则返回false。

some返回的是true或false，来看一下它的返回语句：

```js
return String(row[key]).toLowerCase().indexOf(filterKey) > -1;
```

我们知道传入的key就是name和power，对于每一行name和power的值，判断搜索词是不是它们的子字符串。

- **sort()**：sort这个函数就很常见了，里面传入一个函数定义排序规则。这里我们前面定义的order可以决定是顺序还是逆序。

