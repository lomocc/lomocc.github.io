---
title: ActionScript 3.0 项目中的自加载 Preloader 最佳实现
---
# ActionScript 3.0 项目中的自加载 Preloader 最佳实现

先来看一个示例：[自加载条](http://sample.lomo.cc/flash/preloader/)。

这里我要说的是自加载，因为大家都知道flash要实现加载条无非两种方式，用单独的一个专用加载器外部加载，和自加载。由于外部加载这种方式需要 编译好的预加载.swf文件，但我们许多项目更需要的是能够随时自定义的Preloader，这样能够在不同的项目中随机应变（类似Flex自带的 preloader的实现）。

以前以为要实现自加载很简单，后来经过尝试，发现并不是想象的那样。

因为要实现自加载，就不得不将侦听ProgressEvent.PROGRESS的代码与项目本身的代码混合在一起，最终只会让项目越来越大，并且 如果不经过处理，flash将会加载完所有内容才会运行主程序。做到这一步，这就不得不想到MovieClip边加载边播放的特性，利用这个特性，我们可 以实现不需要加载完整个flash就可以运行我们的加载条程序（和早期时间轴上写代码类似）。

```
package cc.lomo.preloader             
{             
    import flash.display.*;             
                                   
    /**             
     * 功能：SWF自身的加载条&lt;br/&gt;             
     * 用法：在需要实现自加载的类声明前加上&lt;code&gt;[Frame(factoryClass=&quot;cc.lomo.preloader.Preloader&quot;)]&lt;/code&gt;             
     * @author vincent             
     */
    public class Preloader extends MovieClip             
    {             
        /**             
         * 是否本地运行              
         */
        protected var mNative:Boolean;             
        /**             
         * 模拟当前加载量(本地运行时)             
         */
        protected var mIndex:int    = 0;             
        /**             
         * 模拟最大加载量(本地运行时)             
         */
        protected const mMax:int    = 100;             
                                   
        public function Preloader()             
        {             
            addEventListener(&quot;addedToStage&quot;, addedToStageHandler);             
        }             
        protected function addedToStageHandler(e:*):void
        {             
            removeEventListener(&quot;addedToStage&quot;, addedToStageHandler);             
            //如果已经加载完，那估计就是本地运行了，这时候我们只有搞个假的Preloader了             
            mNative = loaderInfo.bytesLoaded == loaderInfo.bytesTotal;             
                                           
            addListeners();             
        }             
        /**             
         *              
         * 侦听加载事件             
         */  
        protected function addListeners():void
        {             
            if(mNative)             
                addEventListener(&quot;enterFrame&quot;, enterFrameHandler);             
            else
            {             
                loaderInfo.addEventListener(&quot;progress&quot;, progressHandler);             
                loaderInfo.addEventListener(&quot;complete&quot;, completeHandler);             
            }             
        }             
        /**             
         *              
         * 移除加载事件             
         */  
        protected function removeListeners():void
        {             
            if(mNative)             
                removeEventListener(&quot;enterFrame&quot;, enterFrameHandler);             
            else
            {             
                loaderInfo.removeEventListener(&quot;progress&quot;, progressHandler);             
                loaderInfo.removeEventListener(&quot;complete&quot;, completeHandler);             
            }             
        }             
        /**             
         * 用ENTER_FRAME模拟加载事件(本地运行时)             
         * @param e             
         *              
         */  
        protected function enterFrameHandler(e:*):void
        {             
            mIndex ++;             
            setProgress(mIndex / mMax);             
            mIndex &gt; mMax &amp;&amp; completeHandler();             
        }             
        /**             
         * 显示进度条             
         * @param value 进度比 0.0 ~ 1.0             
         *              
         */  
        protected function setProgress(value:Number):void
        {             
            graphics.clear();             
            if(value == 1)             
                return;             
            graphics.beginFill(0);             
            graphics.drawRect(0,stage.stageHeight/2, value * stage.stageWidth, 1);             
            graphics.endFill();             
        }             
        /**             
         * 加载事件             
         * @param e             
         *              
         */  
        protected function progressHandler(e:*):void
        {             
            setProgress(loaderInfo.bytesLoaded/loaderInfo.bytesTotal)             
        }             
        protected function completeHandler(e:*=null):void
        {             
            removeListeners();             
                                           
            addEventListener(&quot;enterFrame&quot;, init);             
        }             
        /**             
         * 加载完成后 构造主程序             
         */
        protected function init(e:*):void
        {             
            //currentLabels[1].name 获得第二帧的标签 也就是主程序的类名以&quot;_&quot;连接如：com_adobe_class_Main,我们需要将其转换为com.adobe.class::Main这样的格式             
            var prefix:Array = currentLabels[1].name.split(&quot;_&quot;);             
            var suffix:String = prefix.pop();             
            var cName:String =  prefix.join(&quot;.&quot;) + &quot;::&quot; + suffix;             
            //判断是否存在主程序的类             
            if(loaderInfo.applicationDomain.hasDefinition(cName))             
            {             
                //知道存在主程序的类了，删除enterFrame的侦听             
                removeEventListener(&quot;enterFrame&quot;, init);             
                                               
                var cls:Class = loaderInfo.applicationDomain.getDefinition(cName) as Class;             
                var main:DisplayObject = new cls();             
                parent.addChild( main );             
                parent.removeChild(this);             
            }             
        }                                                    
    }             
}
```

再来看看主程序怎么用的：

```
package
{           
    import flash.display.Sprite;           
    import flash.events.Event;           
                           
    [Frame(factoryClass=&quot;cc.lomo.preloader.Preloader&quot;)]           
    public class Test extends Sprite           
    {           
        public function Test()           
        {           
            super();           
            this.addEventListener(Event.ADDED_TO_STAGE, onadded);           
        }           
                           
        protected function onadded(event:Event):void
        {           
            trace(&quot;真实内容加载到舞台&quot;);           
        }           
    }           
}
```

只需一句代码：

```
[Frame(factoryClass="cc.lomo.preloader.Preloader")]
```

这样就实现了加载条，有兴趣的可以研究一下，就是这么简单。不太喜欢写太多文字，就到这里吧。
