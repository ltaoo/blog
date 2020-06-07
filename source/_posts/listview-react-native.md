---
title: ListView 踩过的坑
categories: React
tags:
- react-native
- listview
---

ListView 组件在 react-native 开发中是会频繁使用的。所以在实际开发时，要实现多选操作时碰到一个坑，虽然使用很奇葩的手段解决了，但并没有从根本上解决遇到的问题。经过别人指点后，终于了解问题产生的原因，记录下这个算得上是基础知识的坑。
<!--more-->

其实只有一个问题，但是这里也将 ListView 常用到的功能点列出来。

## 渲染数据
```javascript
'use strict';
import React, {Component} from 'react';
import {
  StyleSheet,
  View,
  ListView,
  Text,
  Image,
  // width and height of screen
  Dimensions
} from 'react-native';
// 获取屏幕宽高
let {height, width} = Dimensions.get('window');
// 从豆瓣api一次获取多少条数据
let pageSize = 5;

export default class ListViewDemo extends Component {
  constructor(props) {
    super(props);
    // 初始化 state
    this.state = {
      loading: true,
      data: [],
      dataSource: new ListView.DataSource({ rowHasChanged: (r1, r2) => r1 !== r2 })
    };
  }
  // 
  componentDidMount() {
    // 从豆瓣api获取数据，搜索react相关的书籍
    fetch('https://api.douban.com/v2/book/search?q=react&count='+pageSize)
      .then(res=> {
        if(res.status === 200) {
          // 把得到的字符串转换为对象
          let data = JSON.parse(res._bodyInit).books;
          this.setState({
            loading: false,
            data: data,
            dataSource: this.state.dataSource.cloneWithRows(data)
          });
        }
      })
      .catch(err=> {
        alert(JSON.stringify(err));
      })
  }
  // 单列样式
  _renderRow(row, sectionId, rowId) {
    return (
      <View 
        style = {styles.item}
      >
        <Image
          source = {{uri: row.image}}
          style = {{width: 85, height: 120, marginRight: 20}}
          resizeMode = 'stretch'
        />
        <View style = {{flexDirection: 'column', width: width-150}}>
          <Text style = {{fontSize: 16}}>{row.title}</Text>
          <Text
            numberOfLines = {3}
            style = {{color: '#ccc'}}
          >{row.summary}</Text>
        </View>
      </View>
    )
  }
  render() {
    if(this.state.loading) {
      // 如果正在加载数据，就显示 loading...
      return (
        <View style = {styles.empty}>
          <Text>loading...</Text>
        </View>
      )
    }
    return (
      <View style = {styles.container}>
        <ListView
          dataSource = {this.state.dataSource}
          renderRow = {this._renderRow.bind(this)}
     // 由于屏幕高度一次只够显示4条，所以这里设置为4可以稍微提高性能？
          initialListSize = {4}
          // 隐藏滚动条
          showsVerticalScrollIndicator = {false}
        />
      </View>
    )
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#eee'
  },
  empty: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center'
  },
  item: {
    paddingVertical: 10,
    paddingHorizontal: 20,
    borderBottomWidth: 1,
    borderStyle: 'solid',
    borderColor: '#ccc',
    flexDirection: 'row'
  }
});
```
有一点需要注意，由于 _renderRow 是我们自己定义的方法，所以在这个函数里面出现的 this 值需要手动绑定，让 _renderRow 里面的 this 指向我们需要的this。而在 render 函数里面， this 值就是我们需要的，所以`this._renderRow.bind(this)`就可以了。

> 如果某个需要绑定 this 值的方法经常用到，可以在构造函数内绑定，这样就不需要每次调用的时候都 .bind(this) 了。
```javascript
constructor(props){
    //
    this._renderRow = this._renderRow.bind(this);
}
```

## 增加数据
可以用 onEndReached 属性，值是一个函数，当触底时会执行该函数，在这个函数内获取数据。也可以用
renderFooter 在列表底部渲染一个按钮，点击才加载数据。

这里采用第二种：
```javascript
// 渲染底部按钮的函数
_renderFooter() {
    if(this.state.loadingMore) {
      return (
        <View
          style = {{paddingVertical: 10, justifyContent: 'center', alignItems: 'center'}}
        >
          <Text>loading...</Text>
        </View>
      )
    }
    return (
      <TouchableOpacity
        onPress = {this.loadMore.bind(this)}
        style = {{paddingVertical: 10, justifyContent: 'center', alignItems: 'center'}}
      >
        <Text>click it load more</Text>
      </TouchableOpacity>
    )
  }
```
可以看到这里和之前的思路一样，如果正在加载数据，底部按钮就会显示 loading，加载成功后才显示可点击的按钮。
```javascript
// 获取更多数据的函数
loadMore() {
  // 当点击按钮时，就把底部按钮设置为 loading 状态。
    this.setState({
      loadingMore: true
    });
  // 为了实现分页，**初始化 state 时加上一个属性 index，初始值为2**，因为加载更多的时候就是在加载第二页了
    let start = (this.state.index-1)*pageSize;
    fetch(`https://api.douban.com/v2/book/search?q=react&start=${start}&count=${pageSize}`)
      .then(res=> {
        if(res.status === 200) {
          // parse response
          let response = JSON.parse(res._bodyInit).books;
          if(response.length === 0) {
            alert('no data response');
            return;
          }else {
            let oldAry = [...this.state.data];
            let newAry = [...oldAry, ...response];
            this.setState({
              loading: false,
              loadingMore: false,
              data: newAry,
              dataSource: this.state.dataSource.cloneWithRows(newAry),
       index: this.state.index+1
            });
          }
        }
      })
      .catch(err=> {
        alert(JSON.stringify(err));
      })
  }
```

```javascript
_renderRow(row, sectionId, rowId) {
    return (
      <View 
        style = {styles.item}
      >
        <Image
          source = {{uri: row.image}}
          style = {{width: 85, height: 120, marginRight: 20}}
          resizeMode = 'stretch'
        />
        <View style = {{flexDirection: 'column', width: width-150}}>
          <Text style = {{fontSize: 16}}>{row.title}</Text>
          <Text
            numberOfLines = {3}
            style = {{color: '#ccc'}}
          >{row.summary}</Text>
        </View>
      </View>
    )
  }
```
最后是给 ListView 组件加上 renderFooter 属性
```javascript
<ListView
          dataSource = {this.state.dataSource}
          renderRow = {this._renderRow.bind(this)}
          initialListSize = {4}
          showsVerticalScrollIndicator = {false}
          renderFooter = {this._renderFooter.bind(this)}
        />
```

重点是在 loadMore 方法的这部分代码：
```javascript
    let oldAry = [...this.state.data];
    let newAry = [...oldAry, ...response];
```
即新获取到的数据，如何放到已有的数组中，this.state.data 是保存这前五条数据的数组这个毫无疑问，接下来因为我们不能直接改变 state（需要使用setState），所以不能用 push、unshift，所以这里是先取到 this.state.data，然后使用 `...` 展开，新获取到的数组也展开，一起放到数组中，赋给新变量。

## 删除数据
首先是用来删除的方法：
```javascript
delete(row) {
    let oldAry = [...this.state.data];
    let index = oldAry.indexOf(row);
    oldAry.splice(index, 1);
    let newAry = oldAry;
    this.setState({
      data: newAry,
      dataSource: this.state.dataSource.cloneWithRows(newAry)
    });
  }
```
传入 row 作为参数，查询到长按的item在数组中的序列，然后删除，把删除后的数组赋给一个新变量。
可能
```javascript
let newAry = [
    ...oldAry.slice(0, index),
    ...oldAry.slice(index+1)
];
```
这样会更好？暂时都可以，然后添加上长按事件，给每一行添加长按事件：
```javascript
_renderRow(row) {
    return (
      <TouchableOpacity 
        style = {styles.item}
        onLongPress = {this.delete.bind(this, row)}
      >
        <Image
          source = {{uri: row.image}}
          style = {{width: 85, height: 120, marginRight: 20}}
          resizeMode = 'stretch'
        />
        <View style = {{flexDirection: 'column', width: width-150}}>
          <Text style = {{fontSize: 16}}>{row.title}</Text>
          <Text
            numberOfLines = {3}
            style = {{color: '#ccc'}}
          >{row.summary}</Text>
        </View>
      </TouchableOpacity>
    )
  }
```
就可以了，这里要注意的是，需要传入一个参数。

## 选中状态
上面的实现都还算顺利，在实现选中状态，也就是多选功能时踩到了坑。功能点是：一个列表支持多选，然后可以批量删除。先实现选中功能。

选中功能可以用背景色或者图标来区分选中与未选，有两种思路，
- 一个全局数组，选中后将row放到这个数组中，判断row是否在这个数组中来表示是否选中。
- 添加一个字段，这个字段标识是否被选中。

### 全局数组
先说明，这种方式行不通，或者说很难实现，下面是一步一步尝试到无法实现。
先给初始化 state 添加一个 `selectedAry: []`，用来保存选中的 row。然后是 choose 事件：
```javascript
choose(row) {
    // 
    let oldSeletedAry = [...this.state.seletedAry];
    let index = oldSeletedAry.indexOf(row);
    let newSeletedAry = [];
    if(index > -1) {
      // is exist
      oldSeletedAry.splice(index, 1);
      newSeletedAry = oldSeletedAry;
    }else {
      newSeletedAry = [...oldSeletedAry, row];
    }
    this.setState({
      seletedAry: newSeletedAry
    });
  }

```
如果点击的这个row已经存在被选择数组中，就移除，思路和长按删除一样；否则就是添加到被选择数组中。
然后给每一行添加上点击事件，就在 onLongPress 下面加上
```javascript
onPress = {this.choose.bind(this, row)}
```
然后找个地方放这个数组的长度，这样就能很直观看到选择的数量。
在 ListView 组件上方放一个 Text 组件。
```javascript
<Text>{this.state.seletedAry.length+'/'+this.state.data.length}</Text>
```
看看效果，的确是能够正确显示出数量了，还需要给一个状态来区分出是否被选择。在每一行添加一个 Text 组件，如果没有被选择，就显示 NO，如果被选择了，就显示 YES。
```javascript
// 先判断在不在
let index = this.state.seletedAry.indexOf(row);
// 然后是显示是否被选中，在之前的Text 组件下面再添加一个组件
// ...
<Text>{index > -1 ? 'YES' : 'NO'}</Text>
```
OK，页面能正确显示出 NO 了，点击一下，问题来了，被选择数组的长度+1，但是 NO 并没有变成 YES。原因在于，我们做的这些操作，都没有触发到 ListView 重新渲染，setState 的确会触发 render 函数，但是却不会触发 _renderRow 函数啊，所以这里没有变化。OK，问题既然找到了，解决办法就是在 choose 函数内修改 dataSource，以触发 _renderRow 。

```javascript
choose(row) {
    // 
    let oldSeletedAry = [...this.state.seletedAry];
    let index = oldSeletedAry.indexOf(row);
    let newSeletedAry = [];
    if(index > -1) {
      // is exist
      oldSeletedAry.splice(index, 1);
      newSeletedAry = oldSeletedAry;
    }else {
      newSeletedAry = [...oldSeletedAry, row];
    }
    let oldAry = [...this.state.data];
    let newAry = oldAry;
    this.setState({
      seletedAry: newSeletedAry,
      data: newAry,
      dataSource: this.state.dataSource.cloneWithRows(newAry)
    });
  }
```
拿到旧数组，赋给新数组，这样OK吗，答案是NO。这个地方我的理解是，虽然有新数组了，但是数组里面的对象还是原来的对象，内存地址并没有变化。
举个例子
```javascript
let ary = [{
  name: 'ltaoo'
}, {
  name: 'ltooo'
}];

let newAry = [...ary];

newAry[0].name = 'loooo';

console.log(ary);
```
虽然看起来是修改了 newAry 里面对象的值，实际上原数组 ary 的值也发生了改变。所以我们知道了 ListView 判断数组是否发生了改变，要看里面的元素是否发生了变化，而对象需要内存地址不同，才算发生了变化。

那这里如何解决呢？答案是把对象每个都拷贝：
```javascript
let oldAry = [...this.state.data];
    let newAry = oldAry.map(item=> {
      return Object.assign({}, item);
    });
```
虽然的确是触发了 _renderRow 函数（在_renderRow开始alert可以判断是否触发），但是NO 还是 NO，因为 index 的确是 -1 ，经过拷贝后，每一次看到的数组，和上一次都是不同的，即使如果先拷贝，再放到 selectedAry 中去，只能实现只有一个是 YES，点击了另外的，另一个变为 YES，之前为 YES 的变为了 NO。可以仔细思考为什么，这里放出最后挣扎的代码：
```javascript
choose(row) {
    let oldSeletedAry = [...this.state.seletedAry];
    let newSeletedAry = [];
    // 
    let oldAry = [...this.state.data];
    let newAry = oldAry.map(item=> {
      let newItem = Object.assign({}, item);
      if(item===row) {
        let index = oldSeletedAry.indexOf(item);
        if(index > -1) {
          // 存在原数组中
          oldSeletedAry.splice(index, 1);
          newSeletedAry = oldSeletedAry;
        }else {
          newSeletedAry = [...oldSeletedAry, newItem];
        }
      }
      return newItem;
    });


    this.setState({
      seletedAry: newSeletedAry,
      data: newAry,
      dataSource: this.state.dataSource.cloneWithRows(newAry)
    });
  }
```
### 新增字段
在从接口返回数据后，手动添加上一个字段来标识是否被选中。
```javascript
data = data.map(item=> {
            item.isCheck = false;
            return item;
          });
```
然后页面会根据这个字段显示 YES 或者 NO
```javascript
// 点击选中
choose(row) {
    let oldAry = [...this.state.data];
    let index = oldAry.indexOf(row);
    // 对旧数据中的值进行更新
    let newRow = Object.assign({}, row, {
        isCheck: !row.isCheck
    });
    let newAry = [
      ...oldAry.slice(0, index),
      newRow,
      ...oldAry.slice(index+1)
    ];
    this.setState({
      data: newAry,
      dataSource: this.state.dataSource.cloneWithRows(newAry)
    });
  }
```
判断是否在 selectedAry 数组内可以改成直接判断 row.isCheck
```javascript
<Text>{row.isCheck ? 'YES' : 'NO'}</Text>
```
不过显示已选数量还是需要加上，不过不再使用这个数组来做判断。最终的 choose 代码：
```javascript
choose(row) {
    let oldAry = [...this.state.data];
    let index = oldAry.indexOf(row);
    // 对旧数据中的值进行更新
    let newRow = Object.assign({}, row, {
        isCheck: !row.isCheck
    });
    let newAry = [
      ...oldAry.slice(0, index),
      newRow,
      ...oldAry.slice(index+1)
    ];

    let oldSelectedAry = [...this.state.seletedAry];
    let newSelectedAry = [];
    let seletedIndex = oldSelectedAry.indexOf(row);
    if(seletedIndex > -1) {
      oldSelectedAry.splice(seletedIndex, 1);
      newSelectedAry = oldSelectedAry;
    }else {
      newSelectedAry = [...oldSelectedAry, newRow];
    }
    this.setState({
      data: newAry,
      dataSource: this.state.dataSource.cloneWithRows(newAry),
      seletedAry: newSelectedAry
    });
  }
```
### 总结
数组内如果是对象，要修改对象某个属性的值，一定要使用拷贝，才能够触发_renderRow 函数，重新渲染列表。这个了解了，listView 其实就没什么难点了。这里的选中状态功能，可以拓展出修改列表中的值的功能点。


## 真总结
放上最终的完整代码：
```javascript
'use strict';
import React, {Component} from 'react';
import {
  StyleSheet,
  View,
  ListView,
  Text,
  Image,
  // width and height of screen
  Dimensions,
  TouchableOpacity
} from 'react-native';

let {height, width} = Dimensions.get('window');
// 从豆瓣api一次获取多少条数据
let pageSize = 5;
export default class ListViewDemo extends Component {
  constructor(props) {
    super(props);
    // 初始化 state
    this.state = {
      loading: true,
      data: [],
      dataSource: new ListView.DataSource({ rowHasChanged: (r1, r2) => r1 !== r2 }),
      index: 2,
      seletedAry: []
    };
  }

  // 
  componentDidMount() {
    // 从豆瓣api获取数据，搜索react相关的书籍
    fetch('https://api.douban.com/v2/book/search?q=react&count='+pageSize)
      .then(res=> {
        if(res.status === 200) {
          // parse response
          let data = JSON.parse(res._bodyInit).books;
          this.setState({
            loading: false,
            data: data,
            dataSource: this.state.dataSource.cloneWithRows(data)
          });
        }
      })
      .catch(err=> {
        alert(JSON.stringify(err));
      })
  }

  // 单列样式
  _renderRow(row, sectionId, rowId) {
    return (
      <TouchableOpacity 
        style = {styles.item}
        onLongPress = {this.delete.bind(this, row)}
        onPress = {this.choose.bind(this, row)}
      >
        <Image
          source = {{uri: row.image}}
          style = {{width: 85, height: 120, marginRight: 20}}
          resizeMode = 'stretch'
        />
        <View style = {{flexDirection: 'column', width: width-150}}>
          <Text style = {{fontSize: 16}}>{row.title}</Text>
          <Text
            numberOfLines = {3}
            style = {{color: '#ccc'}}
          >{row.summary}</Text>
          <Text>{row.isCheck ? 'YES' : 'NO'}</Text>
        </View>
      </TouchableOpacity>
    )
  }

  loadMore() {
  // 当点击按钮时，就把底部按钮设置为 loading 状态。
    this.setState({
      loadingMore: true
    });

  // 为了实现分页，初始化 state 时加上一个属性 index，初始值为2，因为加载更多的时候就是在加载第二页了
    let start = (this.state.index-1)*pageSize;
    fetch(`https://api.douban.com/v2/book/search?q=react&start=${start}&count=${pageSize}`)
      .then(res=> {
        if(res.status === 200) {
          // parse response
          let response = JSON.parse(res._bodyInit).books;
          if(response.length === 0) {
            alert('no data response');
            return;
          }else {
            let oldAry = [...this.state.data];
            let newAry = [...oldAry, ...response];
            this.setState({
              loading: false,
              loadingMore: false,
              data: newAry,
              dataSource: this.state.dataSource.cloneWithRows(newAry),
              index: this.state.index+1
            });
          }
        }
      })
      .catch(err=> {
        alert(JSON.stringify(err));
      })
  }
  _renderFooter() {
    if(this.state.loadingMore) {
      return (
        <View
          style = {{paddingVertical: 10, justifyContent: 'center', alignItems: 'center'}}
        >
          <Text>loading...</Text>
        </View>
      )
    }
    return (
      <TouchableOpacity
        onPress = {this.loadMore.bind(this)}
        style = {{paddingVertical: 10, justifyContent: 'center', alignItems: 'center'}}
      >
        <Text>click it load more</Text>
      </TouchableOpacity>
    )
  }
  delete(row) {
    let oldAry = [...this.state.data];
    let index = oldAry.indexOf(row);
    oldAry.splice(index, 1);
    let newAry = oldAry;
    this.setState({
      data: newAry,
      dataSource: this.state.dataSource.cloneWithRows(newAry)
    });
  }
  choose(row) {
    let oldAry = [...this.state.data];
    let index = oldAry.indexOf(row);
    // 对旧数据中的值进行更新
    let newRow = Object.assign({}, row, {
        isCheck: !row.isCheck
    });
    let newAry = [
      ...oldAry.slice(0, index),
      newRow,
      ...oldAry.slice(index+1)
    ];
    let oldSelectedAry = [...this.state.seletedAry];
    let newSelectedAry = [];
    let seletedIndex = oldSelectedAry.indexOf(row);
    if(seletedIndex > -1) {
      oldSelectedAry.splice(seletedIndex, 1);
      newSelectedAry = oldSelectedAry;
    }else {
      newSelectedAry = [...oldSelectedAry, newRow];
    }
    this.setState({
      data: newAry,
      dataSource: this.state.dataSource.cloneWithRows(newAry),
      seletedAry: newSelectedAry
    });
  }
  render() {
    if(this.state.loading) {
      // if is loading
      return (
        <View style = {styles.empty}>
          <Text>loading...</Text>
        </View>
      )
    }
    return (
      <View style = {styles.container}>
        <Text>{this.state.seletedAry.length+'/'+this.state.data.length}</Text>
        <ListView
          dataSource = {this.state.dataSource}
          renderRow = {this._renderRow.bind(this)}
          initialListSize = {4}
          // hidden scroller
          showsVerticalScrollIndicator = {false}
          renderFooter = {this._renderFooter.bind(this)}
        />
      </View>
    )
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#eee'
  },
  empty: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center'
  },
  item: {
    paddingVertical: 10,
    paddingHorizontal: 20,
    borderBottomWidth: 1,
    borderStyle: 'solid',
    borderColor: '#ccc',
    flexDirection: 'row'
  }
});
```

然后是 github 地址 https://github.com/ltaoo/react-native-init/blob/master/src/containers/ListViewDemo.js

