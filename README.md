# vue-dome
vue的一些示例

## demo1

demo1实现的是一个简单的用户管理，其实就是对列表的增删改(没有查)，简单得用到了bootstrap，jQuery和vue。

### 页面设计

页面就是一个简单的bootstrap表格。首先最外层是用了一个bootstrap面板(panel)

```html
        <div class="panel panel-primary">
            <div class="panel-heading">
                用户管理
            </div>
```

panel定义面板，primary会让它的样式变成蓝色的，head则是头部，字体会有加粗等基本样式。

然后就是表格了，这里用到的表格也很简单

```html
<table class="table table-striped table-bordered">
    <thead>
        <tr>
            <th>编号</th>
            <th>用户名</th>
            <th>年龄</th>
            ......
        </tr>
    </thead>
    <tbody>
        <tr>
        	<td>
             	<input type="text" class="form-control">
            </td>
			......
        </tr>
    </tbody>
</table>
```

很简单的一个 表格，有一行标题，下面有几个输入框，分别输入你要保存的信息。

这里主要看一下`class`里面的内容。table中的class，striped表示有条纹的样式，bordered表示有边框，这些我以前也接触过。

这次主要学到了\<input\>里面的class="form-control"，`.form-control` 类的 `<input>`、`<textarea>` 和 `<select>` 元素都将被默认设置宽度属性为 `width: 100%`，这个类专门用到表格里，可以让input，select等输入框占满整个单元格。没有的话确实有点丑。

还有一个分页

```html
<nav>
  <ul class="pagination">
    <li class="disabled"><a href="#">&laquo;</a></li>
    <li class="active"><a href="#" >1</a></li>
    <li><a href="#">2</a></li>
    <li><a href="#">&raquo;</a></li>
  </ul>
</nav>
```

`.pagination`代表分页，`.disabled和.active`代表不可选中和选中当前元素，这里选中了第一页，所以前一页就无法点击&+laquo;和&+raquo;分别代表左箭头&laquo;和右箭头&raquo;。注意这里href里面要有#号，我原本是放空=“”，结果无法实现click里面的函数。

### vue的部分

先来看看我们的数据。我们的数据只有在js里面定义的一个data，data其中有一个rows，就是我们要展示的数据。

```json
rows: [
    {
        id: 0,
        name: 'null',
        age: null,
        school: '光明小学',
        remark: 'null'
    },
    {
        id: 1,
        name: '小明',
        age: 18,
        school: '光明小学',
        remark: '三好学生'
    }
]
```

这里第一个是id为0的数据，这是个“假数据”，这个数据永远不会展示出来，为什么会有这个数据呢？因为我在删除数据的时候，发现把数据全删除的话，会连rows这整个数据都没掉，就是data里面没有rows这个字段。所以我放了一个假数据，即使把数据全删除了，也还会有一个数据存在，这个数据没有展示出来，所以用户也就无法删除。

#### 数据展示

```html
<template v-for="(row,index) in rows">
    <tr v-if="row.id != 0 && (index-1) >= (curpage-1)*pagesize && (index-1) < 		  curpage*pagesize">
        <td>{{row.id}}</td>
        <td>{{row.name}}</td>
			....
        <td>
            <a href="#" @click="Edit(row)">编辑</a>
            <a href="#" @click="Delete(row.id)">删除</a>
        </td>
    </tr>
</template>
```

这里第一行用一个for循环拿到每个数据(row)和索引(index)，索引从0开始。

第二行if的判断条件，排除了第一个数据，所以每一个index要减一，curpage当前页和pagesize每页的数量已经在data里面定义了，if的后半部分判断了该页要显示的数据，不在这一页的不显示。

后面有两个a标签，@click是v-on:click的缩写，定义了点击执行的事件——执行相应的函数。

```html
<a v-on:click="doSomething">...</a>

<!-- 缩写 -->
<a @click="doSomething">...</a>
```

#### 表单输入绑定

vue的表单输入绑定很简单，就是在你要输入的地方，比如input，select等元素上加一个属性v-model

```html
<td>
	<input type="text" class="form-control" v-model="rowtemplate.name">
</td>
```

这里v-model的值是rowtemplate.name，rowtemplate是我们在data里面定义的数据，里面有name，age等字段，这些都是我的要输入的数据。v-model会把每个输入框与rowtemplate里面的某一个值绑定，这种绑定是双向的，改变rowtemplate的话，输入框也会相应改变，改变输入框，rowtempla的数值也会改变。

#### 增删改的实现

vue中表单的增删改挺简单的，来看看代码。

```js
Save: function (event) {
    if (this.rowtemplate.id == 0) {
        this.rowtemplate.id = this.rows[this.rows.length - 1].id + 1;
        this.rows.push(this.rowtemplate);
	}

    this.rowtemplate = {
        id: 0,
        name: '',
        age: '',
        school: '',
        remark: ''
    };
},
Edit: function (row) {
	this.rowtemplate = row;
},
Delete: function (id) {
    for (let i = 0; i < this.rows.length; i++) {
        const element = this.rows[i];
        if (element.id == id) {
            this.rows.splice(i, 1);
            break;
        }
    }
}
```

我们先来看看中间的编辑（Edit）因为它最简单，就一行，把当前要编辑的数据，给rowtemplate，我们知道rowtemplate已经和输入框input和select等绑定在一起了，现在rowtemplate的数据已经变成我们要编辑的数据了，所以我们的输入框也会自动变成我们要编辑的数据，这样直接就完成了回显功能，很强大有没有？！

再来看看保存函数（Save），前面判断id是不是等于0，因为我们新增和编辑都是在同一个输入框中，所以如果是新增则要把id自增，而如果是编辑，我们知道这时输入栏是和我们要更改的那一栏绑定的，你输入栏更改的每个字符都会实时更新到你要修改的那一栏，所以编辑就不存在保存的那个步骤，我们要做的只是**解绑**，将输入栏和要修改的那栏解绑，做法就是给rowtemplate绑定另一个数据，这个数据是我们新生成的，里面内容是空的。

删除很简单了，传入的参数是一个id，我们遍历数据，找到那个id的数据，删除它，这里的删除函数式splice()，第一个参数是你要删除的数据的位置，就是数组的下标，第二个参数是你要删除的数量，1代表只删除这个数据，0代表不删除数据。

#### 分页栏

分页栏是我感觉最繁琐的了。

```html
<nav >
<ul class="pagination ">
	<template v-for="page in Math.ceil(rows.length/pagesize)">
        <li @click="PrePage()" id="prepage" v-if="page==1" class="disabled">
            <a href="#">&laquo;</a>
        </li>
        <li v-if="page==1" @click="NumPage(page,$event)" class="active">
            <a href="#">{{page}}</a>
        </li>
        <li v-else @click="NumPage(page,$event)"><a href="#">{{page}}</a></li>
        <li @clilc="NextPage()" id="nextpage" v-if="page == Math.ceil(rows.length/pagesize)">
            <a href="#">&raquo;</a>
        </li>
    </template>
</ul>
</nav>
```

for循环里计算一共有几页，假设有5页，则page就是1，2， 3，4，5。我们第一次显示的时候默认都是第一页，所以前箭头要加disableed，而第一页要加active。前箭头只出现一次，这里用if在page==1的时候显示，第二个li就是在page==1的时候要加active，而其余的在第三个li中用else，就是除了第一页，其他标签页都不需要加active，最后就是后箭头了，再page等于最后一页的时候显示一次。

前箭头和后箭头分别有自己的点击函数，而中间的那些标签都执行同一个点击函数，传入当前页数，和当前的点击对象。

下面来看看具体的函数

```js
PrePage: function (event) {
	$(".pagination .active").prev().trigger("click");
},
NextPage: function (event) {
	$(".pagination .active").next().trigger("click");
},
NumPage: function (num, event) {
    if (this.curpage == num) {
   		return;
    }
    this.curpage = num;
    $(".pagination li").removeClass("active");
    if (event.target.tagName.toUpperCase() == "LI") {
    	$(event.target).addClass('active');
    } else {
    	$(event.target).parent().addClass('active');
    }
    if (this.curpage == 1) {
    	$('#prepage').addClass('disabled');
    } else {
    	$('#prepage').removeClass('disabled');
    }
    if (this.curpage == Math.ceil(this.rows.length / this.pagesize)) {
    	$('#nextpage').addClass('disabled');
    } else {
    	$('#nextpage').removeClass('disabled');
    }
},
```

上一页和下一页的函数很简单，点击上一页就相当于点击当前页的前一页，点击下一页就是当前页的下一页。

而NumPage看着很繁琐，最后两个if就是判断是不是在第一页或者最后一页，如果是，上一页和下一页则不可选中，而前面的就是先移除active，然后根据传入的event判断要在哪个元素上加上active。

好了，这就是第一个例子，因为是第一个例子，所以写得很详细，不过发现很浪费时间，所以以后可能会把重要的拿出来讲。