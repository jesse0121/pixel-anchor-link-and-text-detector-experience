# text-detector-experience
结合实际项目经验谈文本检测

不知不觉已入行一年。回想一年以来，真的是吃了不少苦。从一个机械的外行，转到算法，所有东西都要重新学习，我的天啊，我都不知道自己怎么熬过来的。
记得刚来公司的时候，看到周围的人都比自己年轻，其中不乏很多95后，顿时压力巨大。连SSH连服务器都不会，代码怎么debug？ vim怎么用？linux脚本怎么写？
c++是什么鬼，make? cmake?，不是说好的只用python吗。编译caffe的时候，死活链接不到.so文件，一堆版本的库改了这个环境变量那个又不对了。。。。。。
后面才发现docker这个神器。总之，该碰到的坑一个也没落下。

以前在国企的时候，每天闲得要死，就等着国家批项目。但核电行业自福岛事故后一直惨淡，等了很多年还是这个鬼样。在里面虚度了6年，发现后面
越来越睡不着觉，每天都在想，难道我这辈子就这么过去了吗？有一天和一个腾讯优图的同学吃饭，问他的年收入，当场把我吓了一跳。不行，凭啥，
为什么我清华机械系毕业收入才这么点。。。。不就是搞搞算法嘛，尝试一下吧。就这么莫名奇妙的转了行。当时的心态是，试试看，不行了再求老领导把我捡回去。

人还是逼着才会成长, 入行以后每天脑袋就没停过，看论文，学c++/linux/编译......等基础，最重要的是学用vim写代码，tmux + vim + ptags + ctags，最简单
的工具，熟练了以后发现比任何ide都爽，直接在服务器上写代码，再也不用代码传来传去。牺牲掉所有周末，除了带孩子就是看书，操作，看书，再操作。。。。。
现在基本上别人已经看不出来我是外行了，偶尔还会听到几句大神。。。。

好了，不扯了，讲讲干货。一年以来，一直在搞各类检测和分割的算法，主要落地的场景是文本检测。由于我们这之前的OCR都是用的传统算法，
基于mser那一套，因此基于深度学习的文本检测我算是头一个吃螃蟹的，真的是一切只能靠自己。搞文本检测最大的感受就是：CVPR上那些论文都是啥玩意。。。。
公开的论文大多数是用来刷榜的，而比较权威的数据集大多数是英文，英文的检测和中文的检测难度是差一个数量级的，说实话，对于这些数据集，我开发了无数种算法
去刷，效果都还可以，当然最好的还是pixel-anchor这篇论文了。可气的是，这篇论文CVPR没有中，所以不会开源了。大家如果想刷刷榜，可以尝试去复现一下，如果需要
实际算法落地，那千万别复现，浪费时间。公开的论文在解决实际问题的时候会碰到各种各样的问题，并不能实际拿来用，而只能起到启发思路的作用。一句话：算法的实际落地比
刷榜难多了。以我们趟过的坑为例，pixel-link和PSENet这些基于像素link的算法，不可避免会碰到粘连的问题，实际应用达不到工业级的鲁棒程度，从原理上没法解决。
比如我们在开发银行存单文本检测器的时候，字段内容和字段名称会印重，我们需要把它们分开，像素link绝对解决不了这个问题，只有直接回归的方式可以。

再比如白翔试验室的论文Textboxes, Textboxes++, seglink，我天，什么鬼，在长中文场景实际根本没法用，我们都复现了一遍。。。。。。。不过通过这些复现过程，我们加深了
对文本检测算法的理解，也启发了思路，修炼了内功，最终才能应用到实际当中。其实实际的场景落地，根本没有那么多trick，只有最基本的原理，和对算法的深刻理解。

实际的需求主要印刷体，无非两种，一种结构化的需求，一种通用的需求。结构化的需求主要是识别出每个字段是什么，需要结构化定制。通用就是只要把文本行的内容识别出来
就行了。两种需求走的算法也不是同一个路线。

结构化场景下的难度比较大，有些挨得紧的内容得区分开，只能走直接回归的路线，通过finetune的方式过拟合。直接回归有两种
方式，一种是基于anchor的回归，一种是角点回归和匹配。当然，里面还有很多问题需要解决，比如基于anchor的回归，没有人会用faster rcnn这种算法来检测文字吧，几百个
bboxes ROI想死的心都有了。单步法SSD？你会发现对于长的文本行，框永远不准，好多坑。角点回归和匹配，这个是受到了advance_east的启发，但和它也完全不一样，里面也有很多
坑要踩。由于工作原因，只能提示到这了，该踩的坑只能自己踩一遍。。。。。。。我们的算法在发票OCR的场景下成功落地，达到了工业级的鲁棒性。

通用印刷体场景下，对于印刷体那种长得变态的文本行，CTPN还可以勉强用用，但是速度太慢。方向矫正 + link应该是主流算法了吧。方向矫正和link有多种方式，脑洞一开
其实有多种方式去研究，当然肯定不会是pixel-link和PSENet这种，这个坑大家可以不用趟了，我们帮你们趟过了。强烈推荐一下我们自己开发的pixel-anchor-link算法，只用合成数据训练就能获得极强的泛化能力，详见展示图片。学习最简单的特征，其泛化能力就会很好。pixel方法容易受背景干扰，anchor方法泛化能力不强，两者有机结合，具有超强的泛化能力和抗干扰能力，根本用不着ctpn这种加lstm的操作，lstm很慢。讽刺的是，这种实用的方法根本发不了顶会论文，因为一方面公司不会发实用的方法出去，另一方面学术圈的怪象发论文先benchmark刷刷榜。刷无数的论文-->复现-->没用-->继续刷论文不知道坑了多少人。

自然场景OCR？好像这个应用场景不多，要达到工业级的应用还有一段路要走。

有些细节不方便细说，只能帮大家到这了。愿你不要像我一样趟这么多坑，不过趟坑使人成长，内功最重要，把握分类和回归两个核心，针对实际场景解决实际问题，
有时候越简单越好，算法保佑你。



