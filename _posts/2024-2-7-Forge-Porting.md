---
layout: post
title: Minecraft 1.18/1.19 模组终极移植指南
---

什么？你在玩 1.19 整合包，但最爱的模组却停留在 1.18.2？不妨动动手把它移植到 1.19！

> 本指南只适用于 Forge 模组，对应游戏版本 1.18.X 和 1.19.X 之间互相移植！

## Component 类

Component 类提供了对游戏界面中显示的字符串进行格式化、本地化的功能。在 1.18版本，这些功能被封装在不同的类里，而在 1.19版本，它们都被移到了 Component 类下。

### 1.18
  
```java
import net.minecraft.network.chat.TranslatableComponent;
import net.minecraft.network.chat.TextComponent;

new TranslatableComponent(key, arg1, arg2);
TextComponent.EMPTY;
new TextComponent(text);
```

文档：[TranslatableComponent ⇗](https://nekoyue.github.io/ForgeJavaDocs-NG/javadoc/1.18.2/net/minecraft/network/chat/TranslatableComponent.html) [TextComponent ⇗](https://nekoyue.github.io/ForgeJavaDocs-NG/javadoc/1.18.2/net/minecraft/network/chat/TextComponent.html)

### 1.19

```java
import net.minecraft.network.chat.Component;


Component.translatable(key, arg1, arg2);
Component.empty();
Component.literal(text);
```

文档：[Component ⇗](https://nekoyue.github.io/ForgeJavaDocs-NG/javadoc/1.19.3/net/minecraft/network/chat/Component.html)

</td>
</tr>
</table>