#UILabel的文字显示行数处理
最近在项目中频繁遇到显示多少行文字的需求，有的需求是在什么情况下都只显示1行文字，有的需求是要求显示多少行文字由服务器端控制，有的需求是展示全部文字，有的需求是要最多展示5行文字，不过他们有一个共同的特点就是未显示全的文字用 ... 来代替，遇到了这么多与文字显示行数有关的需求，因此我把UILabel处理显示行数的做了一下总结，下面来具体展示一下
##需求1 只显示一行文字
只显示一行文字其实很好处理，在刚开始学习UILabel的使用时我们已经经常会遇到这种情况，那就是当UILabel的宽度（`XXLabel.frame.size.width`）不能完全容纳所有的文字时，文字就不会全部显示，在width之外的文字将不会显示，而用 ... 来代替，所以说这是一种很好处理的情况。

你需要做的只是：

1. 首先设置好UILabel的frame
2. 把需要显示的文字赋值给UILabel的text属性
3. done

这里还有一种情况就是，当文字的长度其实是不能充满你一开始设置的frame的，而你需要使UILabel的长度适应文字的长度，这时你只需要把第三步的改为`[XXLbel sizeTofit]`即可,我在项目中遇到这种情况是在UILabel之后还需紧跟一个UIImage需要显示，这样设置可以使UILabel的长度根据文字来进行自适应，UIImage可以正常进行布局

##需求2 需要显示全部文字
有时候我们会遇到这样一种情况，在某个ViewController中一些文字不能完全显示，我们点击显示全部，文字则会完全展开，大概就类似于微信朋友圈中点击查看全部，文字就完全显示，这样的情况怎么处理呢？
其实这种情况也比较好处理，我们只需让UILabel完全自适应文字即可

你需要做的是：

1. 首先设置UILabel的frame（其实这里只需要设置UILabel的宽度即可）
2. 然后设置UILabel的numberOfLines属性为0（`XXLabel.numberOfLines = 0`）
3. 使UILabel自适应文字（`[XXLabel sizeToFit]`）

这里需要解释一下numberOfLines = 0，在UILabel.h文件中可以发现苹果是这样解释这个属性的：

	这个属性是用来决定UILabel中文字有几行需要绘制，同时决定当UILabel的sizeToFit方法被调用时该怎么做。numberOfLines的默认值是1，即如果你不设置这个属性的话，UILabel默认只显示一行文字。接下来的一句话需要注意，当这个值被设置为0时代表没有限制，可以显示任意多行，根据你的文字来设置多少行，如果文字的高度超过一行的高度、或者UILabel的高度小于文字的高度，文字就会使用UIlabel的lineBreakMode的属性值来对文字进行删节
	
	
注意到上面一段文字中的最后一段话，我们发现如果文字的高度超过一行的高度、或者UILabel的高度小于文字的高度，文字就会使用UIlabel的lineBreakMode的属性值来对文字进行删节，因此如果你还希望使用...来代表未显示的字符的话，需要将UILabel的lineBreakMode属性设置为NSLineBreakByTruncatingTail

lineBreakMode的类型是一个枚举值，类型是NSLineBreakMode，在这里贴出NSLineBreakMode的定义，可以根据需要使用不同的枚举值

	typedef NS_ENUM(NSInteger, NSLineBreakMode) {
    NSLineBreakByWordWrapping = 0,  // Wrap at word boundaries, default
    NSLineBreakByCharWrapping,	   // Wrap at character boundaries
    NSLineBreakByClipping,		   // Simply clip
    NSLineBreakByTruncatingHead,	// Truncate at head of line: "...wxyz"
    NSLineBreakByTruncatingTail,	// Truncate at tail of line: "abcd..."
    NSLineBreakByTruncatingMiddle	// Truncate middle of line:  "ab...yz"
	} NS_ENUM_AVAILABLE(10_0, 6_0);

##需求3 服务器端控制显示多少行文字

遇到这种需求的时候不要慌，首先需要跟PM详细询问，是服务器控制某个页面所有的UILabel都使用一个值来控制行数显示还是针对每一个UILabel都有一个相应的值来进行控制行数显示。其实这个显示行数的没有必要由服务器来控制，客户端完全可以做这样的事，用不到服务器，服务器只要把需要的显示的文字传回来就行了，这种情况也是因为PM对技术不太属性导致的，所以一定要与PM说清楚这一点，不然有可能你最终写出来的代码需要重写。

好了，现在我们假定PM一定要服务器控制显示多少行，我们也有相应的做法：

1. 设置UILabel的frame（不能只设置宽度，高度也要设置，原因在下面会说到）
2. 从服务器返回的值中解析出需显示的行数
3. 将UILabel的numberOfLines设置为第2步中的行数
4. 对行数进行判断，如果行数为1，就不用sizeToFit（因为numberOfLines = 0时，调用sizeToFit就不会显示...），否则，需要调用`[XXLabel sizeToFit]`


搞定

##需求4 最多显示5行，文字不够5行的有多少行显示多少行
这种需求比较常见，比如说微信朋友圈中，超过多少行就会显示收起按钮，剩下的文字应...来代替，没超过那个行数就有多少行显示多少行，有可能在做社区啊、贴吧啊之类的地方会遇到这样的需求

遇到这样的需求也不要慌，也有相应的解决办法

1. 同样是设置UILabel的Frame（不用设置高度）
2. 计算出完全展示文字需要多少高度，这里需要使用NSString的`boundingRectWithSize:options:attributes:context:`方法，将size的宽度使用UILabel.frame.size.width，高度可以设置一个很大的值（10000），如果你对计算文字显示高度的选项有需求可以将指定的NSStringDrawingOptions类型的值传入option参数（注意这些值可以同时使用），如果你对文字的属性有要求可以将相应的属性传入attribute参数，context参数一般可以传nil(因为我发现传nil就能完成任务，对context参数没有详细了解，挖个坑，以后回来补上)
3. 使用UIFont的lineHeight属性得到每一行的高度
4. 使用第2步中得到的文字高度除以每一行的高度即可得到文字的行数
5. 对第4步中得到的行数进行判断，如果 行数 <= 5,UILabel的numberOfLines属性设置为0，否则UILabel的numberOfLines属性设置为5
6. 调用[XXUILabel sizeToFit]方法

搞定

我写了一个小demo来展示这几种情况，稍后我会把代码上传到GitHub，在写这篇文章的时候我注意到了自己还有好多东西不太明白，比如说UIView的`sizeTofit`方法，比如说`boundingRectWithSize:options:attributes:context:`中context参数，接下来我会多这些东西进行学习，把学习笔记整理出来