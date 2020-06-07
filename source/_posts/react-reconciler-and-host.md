---
title: 记 react-reconcile and host 分享
date: 2020/05/13

---



![image-20200513135648263](./react-reconciler 分享讲稿/image-20200513135648263.png)

Hi，大家好，我这次分享的主题是 `react-reconciler and host`，不知道大家对这两个东西有了解吗？没有最好了（笑），希望这次分享能让大家了解这是什么，怎么使用。



![1](./react-reconciler 分享讲稿/image-20200513140952633.png)

我们先来聊聊 `React`，我们现在开发已经离不开框架了，为什么 `React` 或者 `Vue` 变成主流趋势了呢？或者说 `React` 相比原生、`jQuery` 有什么优点呢？

性能好对吧，为什么性能好呢？因为它有虚拟 `DOM` 加 `Diff` 算法，官方有这么一段话，`React` 使用一些聪明的方法来减少更新页面的昂贵的 `DOM` 操作。

![2](./react-reconciler 分享讲稿/image-20200513141859486.png)

用一个例子简单说明

我们有一份数据要渲染到页面上，之前的做法是根据数据生成 `html` 替换原先的；现在做法是找到具体的 `DOM` 发生了什么改变，然后应用这个改变即可。

这两种方式对浏览器来说区别是非常大的，当然这里大家应该都了解所以就不展开了。

![3](./react-reconciler 分享讲稿/image-20200513142553820.png)

OK，我们接下来来介绍 `reconciler`，`react` 在新版本把更新相关的作为了一个单独的包，就叫 `react-reconciler`，那么具体是哪些内容呢？我们先来看看更新过程

首先是创建更新，我们调用 `setState` 就会创建一个更新，然后经过一系列的函数，把这个更新挂载到发生更新的组件对应的 `fiber` 上，最后调用 `renderRootSync`

第二个步骤是去遍历 `fiber` 树，在遍历的过程中，会更新 `fiber` 以及对比子节点这里对比不是所谓的 `diff` ，具体的后面会讲到；并且，在遍历过程还会筛选出有变化，包括更新、移除、新增的 `fiber` 保存起来。

第五个步骤会进行一些准备工作，包括新增 `DOM` 的初始化、如何更新 `DOM` 的说明。

最后就是执行对 `DOM` 的操作了，更新、移除、新增节点。

前面五个步骤是 `reconciler` 的范畴，最后是属于 `host` 的范畴。

可能对 `fiber` 有些疑问，我们接下来就介绍 `fiber`



![5. fiber](./react-reconciler 分享讲稿/image-20200512170957956.png)

`fiber` 是 `React` 内部的数据结构，可以由  `ReactElement` 生成的。我们用 `jsx` 来描述页面，`jsx` 会变成 `ReactElement` 对吧，每个 `Element` 都会生成一个 `fiber` 

![6. fiber tree](./react-reconciler 分享讲稿/image-20200512171125680.png)

所以我们的页面最终会生成一棵 `fiber` 树，父子节点通过 `child` 字段，兄弟节点通过 `sibling` 字段相连。如果有这样 `jsx`，生成的 `fiber` 树是这样的。`fiberRoot` 是内部创建的。

前面提到更新过程会遍历 `fiber` 树，那是深度优先还是广度优先呢？就是说先遍历兄弟节点，还是先遍历子节点？

答案是先遍历子节点，也就是深度优先，这和前面的第四步「收集有变化的 `fiber` 」有关，这里可以看看实现深度遍历的代码，为了方便看这里省去了一些内容

![7. code of loop fiber tree](./react-reconciler 分享讲稿/image-20200512171813274.png)

遍历就是从第一个开始，然后获取它的子节点，如果存在子节点就继续遍历，否则就调用 `completeUnitOfWork`，该方法会去找是否有兄弟节点，如果没有就看看父节点有没有兄弟节点，如果有就返回，继续遍历。

OK，到这里我们就了解了 `fiber`、`fiber` 树和遍历方式对吧，对这部分有疑问吗？如果没有那我们就继续，

![7](./react-reconciler 分享讲稿/image-20200513144132979.png)

大家肯定注意到，前面很多方法名都和 `work` 相关，`beginWork`、`completeWork` 等等，

其实 `work` 就是 `fiber`，只是换了个名字，更确切的说是 `fiber` 在进行更新过程的语义，`React` 把更新 `fiber` 的过程看作「一件事」，并且是每个 `fiber` 的更新都是一件事

除此之外，在开始做事之前，会先创建一个副本，在这个副本上进行工作，原始内容不变，这其实是借鉴自 `git` 分支的概念，我们每次要开发新功能，是不是先切一个 `feature` 分支，在这个分支上写代码，不影响 `master`，完成后再把 `feature` 分支合并到 `master`，`React` 这里也是一样的。`current` 等同 `master`，`workInProgress` 等同 `feature`。

并且开始工作的 `fiber` 是一个循环结构， `current.alternate === workInProgress`，`workInProgress.alternate === current`。好，我们现在知道了 `fiber` 是什么，知道 `work` 是什么对吧，接下来我们再详细聊聊更新过程

![9. setState](./react-reconciler 分享讲稿/image-20200512174501117.png)

更新的起点是  `setState`，我们通常都是使用该方法来触发更新，该方法会创建一个 `update`，然后挂载到 `fiber.updateQueue` 上，`update.payload` 就是更新的内容。

这部分逻辑没有很复杂的地方吧，唯一要注意的可能是 `expirationTime`，即所谓的过期时间，默认情况它是一个常量，1073741823，该值虽然是一个数字，但在 `react` 是用来表达「优先级」的含义，值越大，优先级越高，就是更新这个 `fiber` 的优先级越高。

由于需要使用 `ConcurrentMode` 才会触发不同的 `expirationTime` 所以这里就不展开了，默认情况下更新优先级都相同。



![10. updateComponent](./react-reconciler 分享讲稿/image-20200512174623435.png)

创建更新后并挂载到 `fiber` 上后，调用一系列的方法遍历 `fiber` 树，遍历过程会调用 `beginWork` ，算是真正开始进行更新的方法吧，该方法内会根据 `fiber` 的 `tag` 调用不同的更新方法，比如 `updateClassComponent`、`updateHostComponent` 等等，`updateClassComponent` 内的逻辑可能会比较多，生命周期就是在这里调用的，还有实例化等操作，具体的可以自己看哈，就不展开了，最后会调用 `render` 方法返回子节点，如果存在子节点就调用 `reconcileChildren` 方法，意为「调和」，可以理解成对比吧，`updateHostComponent` 也是一样的。

![11. reconcileChildren code](./react-reconciler 分享讲稿/image-20200512184839641.png)

`reconcileChildren` 有个分支，看是新建，还是对比，但无论是新建还是对比，都是返回新的 `fiber` 作为子 `fiber`。可以说，`reconcilerChildren` 就是用 `ReactElement` 生成新 `fiber` 的方法。

![11](./react-reconciler 分享讲稿/image-20200513151040172.png)

这里是用之前的例子来说明

第一次调用 `reconcileChildren` 时，`workInProgress` 是 `App` 这个 `fiber`，`newChildren` 是 `render` 调用的结果，用  `workInProgress.child` 和 `div` 这个 `element` 对比，返回新 `fiber`；第二次调用 `workInProgress` 就是这个新 `fiber`，也就是 `div` 这个，然后用它的子节点 `p` 和数组 `Element` 做对比；第三次是 `p0` 这个 `fiber`，第四次是 `p1`。

这里能理解吗

![image-20200513151502251](./react-reconciler 分享讲稿/image-20200513151502251.png)

更具体就是判断下 `newChildren` 类型是什么然后调用对应的方法，比如这里的 `reconcileSingleElement` 和 `reconcileChildrenArray`，但是无论哪个，它们都是会返回 `fiber`，不同之处在于是复用已有的 `fiber` 还是直接用 `element `创建新的 `fiber`。

这里详细讲讲是数组的情况吧，因为这里就涉及到 `key` 的作用了

![image-20200513152036804](./react-reconciler 分享讲稿/image-20200513152036804.png)

这里是直接把代码翻译过来了

这么看可能还是很难理解，那我们用图形来说明，这是一个有新增节点、移除节点的例子。

![image-20200513152110606](./react-reconciler 分享讲稿/image-20200513152110606.png)

上面矩形表示的是 `fiber`，下面圆形表示 `ReactElement`，我们要「对比」这些，就是找出新增和移除

首先是第一个，直接调用 `updateSlot` 返回  `fiber`；然后第二个 `fiber p1` 和下面的 `null`，为什么是 `null` 呢，因为 `hasP1 === false` 了，同样调用 `updateSlot` 但是返回 `null`，所以中断遍历；

中断后，优先看看 `ReactElement` 是不是已经遍历完了，如果遍历完了，就说明剩下的 `fiber` 兄弟节点都是被移除的，但是这里发现还有；然后再看看 `fiber` 是否还有兄弟节点，如果没有就说明剩下的 `ReactElement` 都是新增的，但是这里我们发现还有，所以剩下的 `ReactElement` 不是新增的；

然后，把剩下的 `fiber` 保存到 `Map` 中，就是右上角这个，创建这个 `Map` 优先用 `fiber.key` 作为 `key` ，否则就用 `index`。

然后遍历剩下的 `ReactElement`，调用 `updateFromMap`  创建 `fiber`，在方法里会判断是复用已有的还是创建一个全新的，先是 `index === 2` 这个，发现有，那么就复用之前的，并且从 `Map` 中移除 2 这个；然后是 `index === 3` 这个，发现没有，那就是创建一个全新的。

好，遍历完成了，发现 `Map` 中还剩一个 `index === 1` 这个，那这个就是被删除的了。这样，就清楚了删除和新增的具体是哪些了。

这里能理解吗，下面还有两个例子我们再来看看

![image-20200513152717579](./react-reconciler 分享讲稿/image-20200513152717579.png)

这个例子演示的是 `newIdx === newChildren.length` 的例子，和之前一样，先遍历，但是 `newChildren` 直接遍历完了，然后判断 `newIdx === newChildren.length`，那还剩下的兄弟 `fiber` 就肯定是被移除的对吧，所以直接返回。 

![image-20200513152939274](./react-reconciler 分享讲稿/image-20200513152939274.png)

这个例子类似，先是遍历，然后发现不存在兄弟节点，中断遍历，然后由于不存在兄弟节点了，那剩下的 `newChildren` 肯定都是要用来创建新的 `fiber`。

但是现在只是知道了新增和移除的，那更新的呢？是在 `beginWork` 之后，会调用 `completeWork` 方法，该方法会找出更新的 `fiber`

![image-20200513153426095](./react-reconciler 分享讲稿/image-20200513153426095.png)



在调用 `completeWork` 方法前是调用 `completeUnitOfWork`，该方法是用来收集 `effect` 的，`effect` 就是新增、移除和更新的 `fiber`。

![image-20200513155141064](./react-reconciler 分享讲稿/image-20200513155141064.png)

该方法会在遍历过程，`child === null` 的时候调用，这里大概说明了下，不过我们还是用图形来说明吧，这里假设更新同时存在更新、移除和新增。

我们在遍历到 `p0 fiber` 时，它的 `child === null` 所以会调用 `completeUnitOfWork` 对吧，在该方法内，会先调用一次 `complteWork`，如果它有更新，`effectTag` 就会改变，但是它的 `effectTag` 还是 0，所以忽略该 `fiber`，然后返回它的兄弟节点继续遍历，调用 `beginWork` 方法，同样的，`child === null` ，不过在调用 `completeWork` 后，它的 `effectTag` 变成了 `Update`，所以回到 `complteUnitOfWork` 方法内，会把它自身挂载到父 `fiber` 的 `effect chain` 上，然后继续返回兄弟节点；

接下来就是 `p3`，这个是新增的，道理同上；我们再看看具体怎么保存的吧。

![image-20200513155102628](./react-reconciler 分享讲稿/image-20200513155102628.png)

`p2` 是更新的，先看看父 `fiber` 是否存在 `effect chain`，这时是已经存在的，就是之前遍历过程中移除的 `p1`，早已经挂载到父 `fiber` 上了。我们先看看自身上有没有 `effect` ，发现没有，然后把 `returnFiber.nextEffect` 指向自身，再把 `returnFiber.lastEffect` 指向自身。这样就挂载好了 `p2`。

![image-20200513155603107](./react-reconciler 分享讲稿/image-20200513155603107.png)

然后是 `p3`，和 `p2` 同理，不再赘述。

![image-20200513155617899](./react-reconciler 分享讲稿/image-20200513155617899.png)

关键的地方来了，在处理完 `p3` 后，仍然在当前方法也就是 `complteUnitOfWork` 方法内，用 `returnFiber` 继续调用 `completeWork` 和挂载的逻辑，因为这里有一个 `do while` 循环；

这时 `returnFiber === div`，它调用 `complteWork`，然后和前面逻辑一样，把自身的 `effect chain` 挂载到父 `fiber` 上，此时是有的，所以就挂载上去了。

靠这样，所有的 `effect` 最终会到 `fiberRoot` 上。

到这里其实 `reconciler` 的核心部分已经讲完了，有什么问题吗？

![image-20200513160236432](./react-reconciler 分享讲稿/image-20200513160236432.png)

收集好之后，终于可以对 `DOM` 进行操作了，就是调用 `commitRoot`，根据 `effectTag` 的值执行具体的操作。

![image-20200513160251891](./react-reconciler 分享讲稿/image-20200513160251891.png)

移除、新增和更新。

![image-20200513160314477](./react-reconciler 分享讲稿/image-20200513160314477.png)

![image-20200513160947756](./react-reconciler 分享讲稿/image-20200513160947756.png)

更新稍微麻烦些，它要知道怎么去更新，在前面 `complteWork` 方法内会调用 `updateHostComponent`，这个不同于 `beginWork` 内的，这里会去进行对比，得出要更新的内容，保存在 `updateQueue` 字段上，这是一个数组，每两个元素表示一次更新的内容，比如 `children` 变成 2；`className` 变成 `selected`。

如果确实有更新，也是在这里把 `effectTag` 改成 `Update` 的。

到这里，`react-reconciler` 的内容就真的完了，后面的是 `host` 相关的了。

![image-20200513161206888](./react-reconciler 分享讲稿/image-20200513161206888.png)

`host` 就是所谓的宿主，浏览器是宿主，原生是宿主。

先来看看浏览器，简单来说，浏览器页面，就是对 `DOM` 的图形化展示。

![image-20200513161454722](./react-reconciler 分享讲稿/image-20200513161454722.png)

浏览器使用渲染引擎对 `DOM` 进行渲染，变成图片，就是这样。我们改变 `DOM`，渲染引擎重新渲染，生成新的图片。

改变 `DOM` 可以是手动改，也可以让 `reconciler` 修改，如果是 `reconciler`，连接两者的就是 `hostConfig`，即宿主提供一些方法给 `reconciler` 调用。

所以我们如果想在其他宿主使用 `reconciler`，实现这样一个 `hostConfig` 就可以了。甚至我们可以在 `nodejs` 中使用 `react-reconciler`。

![image-20200513161935968](./react-reconciler 分享讲稿/image-20200513161935968.png)

需要提供平台对象，浏览器是 `HTMLElement`，我们这里就叫 `NodeElement` 好了，分为容器和文本；以及创建 `Node` 的方法、移除 `Node` 的方法、更新 `Node` 的方法和插入 `Node` 的方法。

![image-20200513162214744](./react-reconciler 分享讲稿/image-20200513162214744.png)

 `hostConfig.js` 文件保存上述的方法，为了简单所以这里省略了很多。

![image-20200513162329225](./react-reconciler 分享讲稿/image-20200513162329225.png)

连接方式很简单，调用 `reconciler` 传入 `hostConfig` 就可以了。然后向外暴露一个渲染方法，等同于 `ReactDOM.render`。

![image-20200513162424070](./react-reconciler 分享讲稿/image-20200513162424070.png)



然后我们就可以用 `jsx` 在 `nodejs` 平台写代码了不过我们没有 `babel` 所以只能直接用 `createElement` 了。

调用 `render` 方法后，可以打印看看返回值是什么。然后我们可以再次调用 `render`，触发更新逻辑，返回新的结果。

![image-20200513162749684](./react-reconciler 分享讲稿/image-20200513162749684.png)

这里根据 `jsx` 生成的平台对象实例，等同于 `DOM`。再实现一个渲染引擎将这个对象渲染成图片，就实现了我们的宿主平台。

这里附上示例代码

https://codesandbox.io/s/eloquent-moser-ryh3i?file=/src/index.js



![image-20200513163024514](./react-reconciler 分享讲稿/image-20200513163024514.png)

OK，分享到这里就结束了，我们可以再回顾下之前的内容。



![image-20200513163045472](./react-reconciler 分享讲稿/image-20200513163045472.png)

![image-20200513163055633](./react-reconciler 分享讲稿/image-20200513163055633.png)



至此，全部内容就介绍完了。