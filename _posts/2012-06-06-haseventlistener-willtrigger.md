---
title: hasEventListener 和 willTrigger 的区别
---
# hasEventListener 和 willTrigger 的区别

判断一个事件派发者是否已经注册过了某事件侦听器，该用哪个？

hasEventListener：用于判断当前对象是否真实注册了事件

willTrigger：判断此对象或其任何始祖对象是否注册了事件（对象是否可能触发此事件）。

假如：
父容器是parent,子容器是child：

```
parent.addChild( child );
```
如果parent注册了MouseEvent.CLICK事件。
则：

```
parent.hasEventListener(MouseEvent.CLICK)   =    true;                                                 
parent.willTrigger(MouseEvent.CLICK)        =    true;                                                 
child.hasEventListener(MouseEvent.CLICK)    =    false;                                                 
child.willTrigger(MouseEvent.CLICK)         =    true;
```
如果是child注册了MouseEvent.CLICK事件。
则：

```
parent.hasEventListener(MouseEvent.CLICK)   =    false;  
parent.willTrigger(MouseEvent.CLICK)        =    false;                                                 
child.hasEventListener(MouseEvent.CLICK)    =    true;                                                 
child.willTrigger(MouseEvent.CLICK)         =    true;
```  
如果是都没有注册事件，那么都为false了。
