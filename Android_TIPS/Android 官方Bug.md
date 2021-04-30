# Android 官方Bug

- PopupWindow 中的 EditText 无法复制/粘帖.TextView 无法选择.
  
为什么PopupWindow里面的EditText无法粘贴:

PopupWindow 内的 View 是无法获取 WindowToken 的，而粘贴功能也是一个PopupWindow，它的显示必定需要 WindowToken，因此无法粘贴。
[google issue][https://issuetracker.google.com/issues/36984016]
