---
comments: true
---

## 妙控鼠标调速

如果你感觉鼠标指针移动很慢，可以通过如下设置来提高鼠标的移动速度。


=== "终端"
```bash
# 查看当前鼠标速度，这时候电脑一般会输出3，这是电脑默认的速度。
defaults read -g com.apple.mouse.scaling

# 调整速度，在终端中输入如下命令。
defaults write -g com.apple.mouse.scaling 7
```

以上代码中最后的数字 `7` 是我们新设置的速度，一般建议将该数值设置为 6～8，我设置的是 `7`，感觉刚刚好。