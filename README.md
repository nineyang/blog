#  如何更优雅的给控制器“减负”

Published on Mar 21, 2018 in
[Listen](https://www.hellonine.top/index.php/category/listen/)[PHP](https://www.hellonine.top/index.php/category/PHP/)
with  0 comment

[PHP](https://www.hellonine.top/index.php/tag/PHP/)
[Listen](https://www.hellonine.top/index.php/tag/Listen/)

`MVC`是一个非常伟大的概念，但是最近我发现一个现象，包括我自己，我们在最开始接触`MVC`概念时，我们非常严谨地贯彻这种分层思想，`Controller`层处理业务逻辑，而`Model`层只是单纯的处理数据`I/O`。但是，伴随着我们项目体量的逐渐增大，控制器的负担也越来越大。这样一来会有一个非常明显的弊端，当我们在定位`BUG`时，我们总是需要对照着代码查看许久。除此之外，彼此的业务代码并没有太好的关联，这使得我们想要抽出一个`Service`时就显得极为困难。
因此，是时候给我们的控制器做一些“减负”了。这里的减负并不意味着会违背`MVC`的设计思想，而是把我们的控制器层的业务适当的分给其他部分。
有使用过一些主流框架的朋友应该都知道，其实很多框架都给`Controller`层做了一些“减负”的工作，比如`KOA`里面的`middleware`，抑或是`Laravel`里面的`Event`,`Policy`等。
但是事与愿违，即使这些框架提供了这些帮助，但是许多人在实际项目中使用到的却很少，当然，也有可能是我接触到的代码不够多。究其原因，窃以为尚未意识到这种理念的重要性。因此，我在这里总结了自己这些年来“减负”的一些经验，同时我也会配合一些代码予以解释。当然，我所写的未必全对，因此希望有幸看到的读者能保持自己的独立性。

## 向`Model`分流

我们在写代码时往往会有这样一种场景，我们需要对从`Model`取出来的数据进行加工，但是，加工数据的部分我们经常会放到控制器，毕竟这属于业务逻辑，确实无可厚非，如下伪代码所示:



    // controller
    public function userList()
    {
        $users = array_map(function ($user){
    //        这里会对我们的代码进行业务逻辑的加工
            $user['created_at'] = date('Y-m-d' , $user['created_at']);
            // ...
            return $user;
        } ,$model->availableList());
    }

    // model
    public function availableList()
    {
        // 从数据库取数据
        return $users;
    }

但是我们有没有考虑过这样一个问题，当我们同事来接手我们项目或者我们`debug`时，我们需要了解的代码量非常大，特别是涉及到一些数据加工的格式问题，我们并不需要关心。或者换个角度，当我们遇到数据加工的`bug`时，我们能第一联想到这段代码是放在`Model`层时，是不是更加快捷呢？



    // controller
    public function userList()
    {
        $users = $model->availableList();
    //    处理其他逻辑
    }

    // model
    public function availableList()
    {
        // 从数据库取数据 $users
        return array_map(function ($user){
    //        这里会对我们的代码进行业务逻辑的加工
            $user['created_at'] = date('Y-m-d' , $user['created_at']);
            // ...
            return $user;
        } , $users);
    }

如上代码所示，在`Model`层中已经帮我们封装好了我们所需要的数据以及其格式，当我们在浏览他人代码时，我们并不需要关心他的格式是怎么加工的，我们只需要根据他对方法的命名就能知道是获取的怎样的数据。

## 分离`Controller`

在写具体的方法之前，我想要阐述的一点是，我们在写代码的时候需要保持一定的前瞻性。什么意思？虽然我们的大部分工作都是跟具体的业务逻辑打交道，但是我们经常会发现总会有重复的工作，那么有的人会直接把这段代码复制。但是，在我们复制之前，我们是不是可以问自己这样一个问题：如果接下来还有类似的业务，我们还是复制吗？我们是不是可以把这段基于我们项目的代码抽象出一个`Service`呢？
我举个例子，比如一个网站，可能会有打赏功能，可能也有付费阅读功能，我们不难发现，这两种付费有着相似的地方，比如创建本平台订单系统的业务逻辑，再比如回掉时可能存在的相同业务逻辑，所以这段代码我们是不是可以以一个`trait`的形式做一个`Service`。



    trait PayService
    {
        private $_callback = null;

        public function createOrder()
        {
    //        处理你的业务逻辑，配置调用三方支付接口的参数等
        }

        public function callback()
        {
    //        处理共同的回调逻辑

            $this->handler();
        }
    }

这里我们保留了一个`handler`方法来处理每个功能独有的业务逻辑，至此，我们就可以非常方便的扩展我们的支付服务了。

给控制器减负的方法还有很多，比如对我们加工数据的部分，其实我们也可以不放到`Model`，我们也可以单独开辟一层来处理我们的数据加工。让控制器变得清晰明朗，每个人阅读代码时都能非常快速的了解控制器下的每个方法在处理什么业务逻辑。这便是我们给控制器减负的目的。
我很喜欢「包」的概念和设计思想，当我们在使用包时，不仅仅意味着方便，更重要的是，他做为一个独立的“组件”存在于我们的代码逻辑中，与我们项目的代码不存在任何的耦合，同时我们也无需知道他的具体实现。
