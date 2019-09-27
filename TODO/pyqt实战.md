看完QT designer教程   开发最简单的平行优惠 (仿照demo)







### 1. 自定义事件函数

1. 点击编辑槽
2. 在按钮上拖动, 指向你要关联的控件.
3. 选择一个事件, 如 clicked
4. 在右边点击Edit, 然后再点+号, 输入你的函数
5. 可以看到你的函数了

https://stackoverflow.com/a/7965081



### 2. 自定义继承控件

1. 在控件上右键 Promote to...
2. 设置 "Promoted class name" 为 "LineEdit"
3. 设置 "Header file" 为 module 路径 (e.g. `myapp.LineEdit`). 

https://stackoverflow.com/a/22543079











You can only add custom signals/slots to *subclasses* of Qt classes.

As a demonstration of this, make a connection between pushButton and the top-level widget. When the connection dialog is shown, you will see that the right-hand edit button is now enabled. This is because the top-level widget will usually be a subclass of QWidget, QMainWindow, or QDialog that is defined by the application.

To add custom signals/slots to *child* widgets, you would need to use [widget promotion](http://doc.qt.io/qt-4.8/designer-using-custom-widgets.html#promoting-widgets), so that you can specify a subclass that will be supplied by your application. See [this answer](https://stackoverflow.com/a/22543079/984421) for how to promote widgets in PyQt.





















### ui 和逻辑分离







*if* __name__ == '__main__':

​    *import* sys

​    app = QtWidgets.QApplication(sys.argv)

​    MainWindow = QtWidgets.QMainWindow()

​    ui = Ui_MainWindow()

​    ui.setupUi(MainWindow)

​    MainWindow.show()

​    sys.exit(app.exec_())





### 参考资料

+ https://zhuanlan.zhihu.com/p/48373518
+ https://zhuanlan.zhihu.com/p/75673557