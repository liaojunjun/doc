# 键盘：keydown 和 keyup

在我们开始学习键盘的相关内容之前，请注意，在现代设备上，还有其他“输入内容”的方法。例如，人们使用语音识别（尤其是在移动端设备上）或用鼠标复制/粘贴。

因此，如果我们想要跟踪 `<input>` 字段中的所有输入，那么键盘事件是不够的。无论如何，还需要一个名为 `input` 的事件来跟踪 `<input>` 字段中的更改。对于这样的任务来说，这可能是一个更好的选择。稍后我们将在[事件：change，input，cut，copy，paste ]一章中介绍它们。

当我们想要处理键盘行为时，应该使用键盘事件（虚拟键盘也算）。例如，对方向键 `key:Up` 和 `key:Down` 或热键（包括按键的组合）作出反应。

## 测试台 [#keyboard-test-stand]

<iframe data-src="https://liaojunjun.github.io/nice/root/event-details/keyboard-events/keyboard-dump.view/index.html" width="100%" height="400"></iframe>

## Keydown 和 keyup

当一个按键被按下时，会触发 `keydown` 事件，而当按键被释放时，会触发 `keyup` 事件。

### event.code 和 event.key

事件对象的 `key` 属性允许获取字符，而事件对象的 `code` 属性则允许获取“物理按键代码”。

例如，同一个按键 `key:Z`，可以与或不与 `Shift` 一起按下。我们会得到两个不同的字符：小写的 `z` 和大写的 `Z`。

`event.key` 正是这个字符，并且它将是不同的。但是，`event.code` 是相同的：

| Key           | `event.key` | `event.code` |
| ------------- | ----------- | ------------ |
| `key:Z`       | `z`（小写） | `KeyZ`       |
| `key:Shift+Z` | `Z`（大写） | `KeyZ`       |

如果按键没有给出任何字符呢？例如，`key:Shift` 或 `key:F1` 或其他。对于这些按键，它们的 `event.key` 与 `event.code` 大致相同：

| Key             | `event.key` | `event.code`                |
| --------------- | ----------- | --------------------------- |
| `key:F1`        | `F1`        | `F1`                        |
| `key:Backspace` | `Backspace` | `Backspace`                 |
| `key:Shift`     | `Shift`     | `ShiftRight` 或 `ShiftLeft` |

请注意，`event.code` 准确地标明了哪个键被按下了。例如，大多数键盘有两个 `key:Shift` 键，一个在左边，一个在右边。`event.code` 会准确地告诉我们按下了哪个键，而 `event.key` 对按键的“含义”负责：它是什么（一个 "Shift"）。

假设，我们要处理一个热键：`key:Ctrl+Z`（或 Mac 上的 `key:Cmd+Z`）。大多数文本编辑器将“撤销”行为挂在其上。我们可以在 `keydown` 上设置一个监听器，并检查哪个键被按下了。

这里有个难题：在这样的监听器中，我们应该检查 `event.key` 的值还是 `event.code` 的值？

一方面，`event.key` 的值是一个字符，它随语言而改变。如果访问者在 OS 中使用多种语言，并在它们之间进行切换，那么相同的按键将给出不同的字符。
因此检查 `event.code` 会更好，因为它总是相同的。

像这样：

```js
document.addEventListener("keydown", function (event) {
  if (event.code == "KeyZ" && (event.ctrlKey || event.metaKey)) {
    alert("Undo!");
  }
});
```

## 自动重复

如果按下一个键足够长的时间，它就会开始“自动重复”：`keydown` 会被一次又一次地触发，然后当按键被释放时，我们最终会得到 `keyup`。因此，有很多 `keydown` 却只有一个 `keyup` 是很正常的。

对于由自动重复触发的事件，`event` 对象的 `event.repeat` 属性被设置为 `true`。

## 默认行为

默认行为各不相同，因为键盘可能会启动许多可能的东西。

例如：

- 出现在屏幕上的一个字符（最明显的结果）。
- 一个字符被删除（`key:Delete` 键）。
- 滚动页面（`key:PageDown` 键）。
- 浏览器打开“保存页面”对话框（`key:Ctrl+S`）
- ……等。

阻止对 `keydown` 的默认行为可以取消大多数的行为，但基于 OS 的特殊按键除外。例如，在 Windows 中，`key:Alt+F4` 会关闭当前浏览器窗口。并且无法通过在 JavaScript 中阻止默认行为来阻止它。

例如，下面的这个 `<input>` 期望输入的内容为一个电话号码，因此它不会接受除数字，`+`，`()` 和 `-` 以外的按键：

```html
<script>
  function checkPhoneKey(key) {
    return (
      (key >= "0" && key <= "9") ||
      key == "+" ||
      key == "(" ||
      key == ")" ||
      key == "-"
    );
  }
</script>
<input
  onkeydown="return checkPhoneKey(event.key)"
  placeholder="tel"
  type="tel"
/>
```

请注意，像 `key:Backspace`，`key:Left`，`key:Right`，`key:Ctrl+V` 这样的特殊按键在输入中无效。这是严格过滤器 `checkPhoneKey` 的副作用。

让我们将过滤条件放松一下：

```html
<script>
  function checkPhoneKey(key) {
    return (
      (key >= "0" && key <= "9") ||
      key == "+" ||
      key == "(" ||
      key == ")" ||
      key == "-" ||
      key == "ArrowLeft" ||
      key == "ArrowRight" ||
      key == "Delete" ||
      key == "Backspace"
    );
  }
</script>
<input
  onkeydown="return checkPhoneKey(event.key)"
  placeholder="Phone, please"
  type="tel"
/>
```

现在方向键和删除键都能正常使用了。

……但我们仍然可以使用鼠标右键单击 + 粘贴来输入任何内容。因此，这个过滤器并不是 100% 可靠。我们可以让它就这样吧，因为大多数情况下它是有效的。或者，另一种方式是跟踪 `input` 事件 —— 在任何修改后触发。这样我们就可以检查新值，并在其无效时高亮/修改它。

## 遗存

过去曾经有一个 `keypress` 事件，还有事件对象的 `keyCode`、`charCode` 和 `which` 属性。

大多数浏览器对它们都存在兼容性问题，以致使该规范的开发者不得不弃用它们并创建新的现代的事件（本文上面所讲的这些事件），除此之外别无选择。旧的代码仍然有效，因为浏览器还在支持它们，但现在完全没必要再使用它们。
