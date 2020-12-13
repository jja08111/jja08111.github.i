---
title: "[Flutter] One Ui Scroll View"
date: 2020-12-13 19:14:00 -0400
categories: flutter
tags:
- flutter
- nested_scroll_view
- custom
---

Let's make appBar like samsung ONE UI style animation in flutter.

![one_ui_scroll_view_result](/assets/images/one_ui_scroll_view_result.gif)

# Simple use

It is simple to use. See the example code.

```dart
class ExamplePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: OneUiScrollView(
        expendedTitle: Text('ONE UI SCROLL VIEW', style: TextStyle(fontSize: 32)),
        collapsedTitle: Text('Home', style: TextStyle(fontSize: 24)),
        actions: [
          IconButton(icon: const Icon(Icons.more_vert), onPressed: () {}),
        ],
        children: List<Widget>.generate(9, (index) {
          return Padding(
            padding: const EdgeInsets.all(16),
            child: Text(
              'List ${index + 1}',
              style: TextStyle(fontSize: 24),
            ),
          );
        }),
      ),
    );
  }
}
```



# In more detail

The space of `expandedTitle` is setted by 0.381 ratio on screen. And when a user scrolling is end, **appBar snaps to expand or collapse**. But scroll gesture isn't perfect yet. 

You can customize `appBarColor`, `bottomDivider`, `actions`, and also `expandedHeight' but I recommend using default height.

# Github

Welcome to visit [github/one_ui_scroll_view](https://github.com/jja08111/one_ui_scroll_view) that is uploaded whole code. 
