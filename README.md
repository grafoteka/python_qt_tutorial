# How to quickly make a Qt GUI with Python

## Requirements
Qt designer and python Qt bindings:

    sudo apt-get install qt4-designer python-qt4

iPython is optional but recommended:

    sudo apt-get install ipython

You may want a proper editor for Python, here you have a guide on how to setup **Sublime Text** or **PyCharm** [My development environment](https://gist.github.com/awesomebytes/c932b2502fab32d0c8bb).

## Design the GUI
Fire up Qt designer

    designer-qt4

Choose `Main Window` on the New form dialogue (File > New...).

![New form dialogue](screenshots/qtdesigner1.png)

Add a `PushButton` and a `Label` from the left column menu (Widget Box). Change the name of the elements on the top right menu (Object Inspector), `my_button` and `my_label` are good enough names.

![Add elements](screenshots/qtdesigner2.png)

Save the file as `my_gui.ui`.

## Script the behaviour
Open your favorite editor (I recommend Sublime Text with Anaconda IDE plugin) and paste:

````python
#!/usr/bin/env python

import sys
from PyQt4 import QtGui, uic


class MyWindow(QtGui.QMainWindow):
    def __init__(self):
        super(MyWindow, self).__init__()
        uic.loadUi('my_gui.ui', self)
        self.show()


if __name__ == '__main__':
    app = QtGui.QApplication(sys.argv)
    myWindow = MyWindow()
    app.exec_()

````

Give the file executable permissions:

    chmod +x my_gui.py

And we can execute it 

     ./my_gui.py

and you have a GUI showing stuff!

![First GUI](screenshots/my_gui.png)

But we want the GUI to do stuff. So we need to connect buttons to actions.

Once you learn how Qt works you can just google for methods (the [official Qt4 documentation](http://doc.qt.io/qt-4.8/) is for C++ but the methods are the same in Python) and so and you'll end up with something like:

````python
#!/usr/bin/env python

import sys
from PyQt4 import QtGui, uic


class MyWindow(QtGui.QMainWindow):
    def __init__(self):
        super(MyWindow, self).__init__()
        uic.loadUi('my_gui.ui', self)
        self.my_button.clicked.connect(self.on_my_button_push)
        self.show()

    def on_my_button_push(self):
        self.my_label.setText("Button Pushed!")


if __name__ == '__main__':
    app = QtGui.QApplication(sys.argv)
    myWindow = MyWindow()
    app.exec_()
````

But if you want autocompletion to know what methods and properties has every element you can either:
* Use iPython, load the code and use the dynamic introspection to check stuff (my preferred option)
* Generate a Python class using `pyuic.py` and load it in your code instead of using `uic.loadUi` (a bit cumbersome)

See `Extras` for more on that.

# Examples

## Run a shell command and retrieve its output
See `command_runner/command_runner.py`.

![Command runner](screenshots/command_runner.png)

## Show a loading bar meanwhile doing something
See `command_runner/command_runner_progressbar.py` for an extension of `command_runner/command_runner.py` that shows a progress bar while running a command.

![Command runner progressbar](screenshots/command_runner_progressbar.png)

## Show Error/Warning popup
See `other_examples/error_window.py`.

![Error window](screenshots/error_window.png)

## Choose a file
See `other_examples/choose_file.py`.

![Choose file](screenshots/choose_file.png)

## Choose a folder
See `other_examples/choose_folder.py`.

![Choose folder](screenshots/choose_folder.png)

## Using a dropdown/combo_box
See `other_examples/generate_dropdown.py`.

![Dropdown example](screenshots/dropdown.png)

## Generate dynamically GUI fields
See `other_examples` folder, they aren't using any .ui file.

## Real time plot
TODO

## Create a rqt_gui plugin
TODO


# Extras
Sublime Text with Anaconda plugin or PyCharm (and probably other editors) will autocomplete for you some stuff but anything that is dynamically generated will not be able to autocomplete it (for example whatever you loaded from a .ui file). Here I present a couple of ways of getting autocompletion on that.

## Use iPython for autocompletion
Fire up iPython

    ipython

You must take into account that you want to remain in the iPython shell so you must comment the line `app.exec_()` (or in the next step deleting it).

And load your file

    %load my_gui.py

And after hitting enter again, you'll see the window and you'll be able to do introspection about everything available.

A trick for not needing to do this commenting/deleting line is to modify the code with:
````python
#!/usr/bin/env python

import sys
from PyQt4 import QtGui, uic


def running_in_ipython():
    """Check if the interpreter is iPython as when debugging we don't want to
    execute app.exec_() to remain in the shell to do dynamic introspection"""
    try:
        __IPYTHON__
        return True
    except NameError:
        return False


class MyWindow(QtGui.QMainWindow):
    def __init__(self):
        super(MyWindow, self).__init__()
        uic.loadUi('my_gui.ui', self)
        self.my_button.clicked.connect(self.on_my_button_push)
        self.show()

    def on_my_button_push(self):
        self.my_label.setText("Button Pushed!")


if __name__ == '__main__':
    app = QtGui.QApplication(sys.argv)
    myWindow = MyWindow()
    if not running_in_ipython():
        app.exec_()

````


## Generate a Python class to help on autocompletion
First locate your `pyuic.py` file

    locate pyuic.py

In my case it's at `/usr/lib/python2.7/dist-packages/PyQt4/uic/pyuic.py`.

Execute it with your ui file

    python /usr/lib/python2.7/dist-packages/PyQt4/uic/pyuic.py my_gui.ui > my_gui_class.py

And now your code should be changed to:

````python
#!/usr/bin/env python

import sys
from PyQt4 import QtGui
from my_gui_class import Ui_MainWindow


class MyWindow(QtGui.QMainWindow):
    def __init__(self):
        super(MyWindow, self).__init__()
        self.ui = Ui_MainWindow()
        self.ui.setupUi(self)
        self.ui.my_button.clicked.connect(self.on_my_button_push)
        self.show()

    def on_my_button_push(self):
        self.ui.my_label.setText("Button pushed!")


if __name__ == '__main__':
    app = QtGui.QApplication(sys.argv)
    myWindow = MyWindow()
    app.exec_()

````

Now you'll get all the possible autocompletion and documentation available (won't be everything).

You'll need to recreate the `my_gui_class.py` every time you change the `my_gui.ui` file.
