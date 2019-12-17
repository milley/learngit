# FileZilla远程路径中文显示乱码

在FTP服务器使用FileZilla连接，远程中文路径显示乱码，通过站点管理中的字符集标签页设置：使用自定义的字符集->编码：GB2312

# MFC自动化测试

[pywinauto](http://pywinauto.github.io/)

## 1. 根据标题获取已经打开的窗口

```python
app = Application().connect(title=r"登陆-xxx")
```

## 2. 获取所有的控件

```python
dlg = app.top_window()
print(dlg.print_control_identifiers()
```

## 3. 填入text文本框

```python
dlg[u'工号：Edit'].set_text('xxx')
dlg[u'密码：Edit'].set_text('xxx')
```

## 4. 点击按钮登陆

```python
dlg['Button3'].click()
```
