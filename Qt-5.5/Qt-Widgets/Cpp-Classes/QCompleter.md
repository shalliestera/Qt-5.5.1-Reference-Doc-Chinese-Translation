# QCompleter Class #
[Qt 5.5] > [Qt Widgets] > [C++ Classes] > QCompleter

## 目录 ##
* Public Types
* Properties
* Public Functions
* Public Slots
* Signals
* Reimplemented Protected Functions
* Detailed Description
* Basic Usage
* Iterating Through Completions
* The Completion Model
* Handling Tree Models

[QCompleter]类提供基于item模型的完成功能。[More...]()
Header:	#include <QCompleter>
qmake:	 QT += widgets
Since:	 Qt 4.2
Inherits:	[QObject]

[所有成员（包括继承成员）](./qcompleter-members.md)

## 公共类型 ##

enum	[CompletionMode]() { PopupCompletion, InlineCompletion, UnfilteredPopupCompletion }
enum	[ModelSorting]() { UnsortedModel, CaseSensitivelySortedModel, CaseInsensitivelySortedModel }

## 属性 ##

> * caseSensitivity : Qt::CaseSensitivity
> * completionColumn : int
> * completionMode : CompletionMode
> * completionPrefix : QString
> * completionRole : int
> * filterMode : Qt::MatchFlags
> * maxVisibleItems : int
> * modelSorting : ModelSorting
> * wrapAround : bool

* 继承自[QObject]()的1项属性

## 公有函数 ##

| 返回值 | 函数名和参数 |
| -----: | :----------- |
|  |QCompleter(QObject * parent = 0) |
|  | QCompleter(QAbstractItemModel * model, QObject * parent = 0)
|  | QCompleter(const QStringList & list, QObject * parent = 0)
|  | ~QCompleter()
| Qt::CaseSensitivity	| caseSensitivity() const
| int	| completionColumn() const
| int	| completionCount() const
| CompletionMode	| completionMode() const
| QAbstractItemModel *	| completionModel() const
| QString	| completionPrefix() const
| int	 | completionRole() const
| QString	| currentCompletion() const
| QModelIndex	| currentIndex() const
| int	| currentRow() const
| Qt::MatchFlags	| filterMode() const
| int	| maxVisibleItems() const
| QAbstractItemModel *	| model() const
| ModelSorting	| modelSorting() const
| virtual QString	| pathFromIndex(const QModelIndex & index) const
| QAbstractItemView *	| popup() const
| void	| setCaseSensitivity(Qt::CaseSensitivity caseSensitivity)
| void	| setCompletionColumn(int column)
| void	| setCompletionMode(CompletionMode mode)
| void	| setCompletionRole(int role)
| bool	| setCurrentRow(int row)
| void	| setFilterMode(Qt::MatchFlags filterMode)
| void	| setMaxVisibleItems(int maxItems)
| void	| setModel(QAbstractItemModel * model)
| void	| setModelSorting(ModelSorting sorting)
| void	| setPopup(QAbstractItemView * popup)
| void	| setWidget(QWidget * widget)
| virtual QStringList	| splitPath(const QString & path) const
| QWidget *	| widget() const
| bool	| wrapAround() const

* 31个继承自[QObject]()的公有函数

## 公有槽 ##

| 返回类型 | 函数名和参数 |
| -------: | :----------- |
| void	| complete(const QRect & rect = QRect()) |
| void	| setCompletionPrefix(const QString & prefix) |
| void	| setWrapAround(bool wrap) |

* 1个继承自[QObject]()的槽

## 信号 ##

| 返回类型 | 函数名和参数 |
| -------: | :----------- |
| void	| activated(const QString & text) |
| void	| activated(const QModelIndex & index) |
| void	| highlighted(const QString & text) |
| void	| highlighted(const QModelIndex & index) |

* 2个继承自[QObject]()的信号

## 重新实现的保护函数 ##

| 返回类型 | 函数名和参数 |
| -------: | :----------- |
| virtual bool	| event(QEvent * ev) |
| virtual bool	| eventFilter(QObject * o, QEvent * e) |

* 9个继承自[QObject]()的函数

### 额外的继承成员 ###
* 1个继承自[QObject]的公有变量
* 10个继承自[QObject]的静态公有成员
* 9个继承自[QObject]的保护函数
* 2个继承自[QObject]的保护变量

## 详细说明 ##

[QCompleter]类提供基于一个item模型的完成 功能。

可以在任何Qt widget内使用[QCompleter]提供自动补全功能，例如[QLineEdit]和[QComboBox]。
当用户开始输入一个单词，[QCompleter]以一个单词列表为基础，提供可能的单词补全。
单词列表作为一个[QAbstractItemModel]提供。
（对于单词列表是静态的简单程序，你可以传递一个[QStringList]给[QCompleter]的构造函数。）

### 基本用法 ###

典型用法，[QCompleter]与[QLineEdit]或[QComboBox]一起使用。举个例子，如何在[QLineEdit]中提供自动补全功能：
```c++
QStringList wordList;
wordList << "alpha" << "omega" << "omicron" << "zeta";

QLineEdit *lineEdit = new QLineEdit(this);

QCompleter *completer = new QCompleter(wordList, this);
completer->setCaseSensitivity(Qt::CaseInsensitive);
lineEdit->setCompleter(completer);
```

[QFileSystemMode]可被用来提供文件名自动补全功能。举个例子：
```c++
QCompleter *completer = new QCompleter(this);
completer->setModel(new QDirModel(completer));
lineEdit->setCompleter(completer);
```

通过调用[setModel()]来设置[QCompleter]对哪个模型起作用。
默认的，[QCompleter]将试图匹配[completion prefix]()（也就是用户开始输入的单词）和模型中存储的第0列的[Qt::EditRole]数据。
这种行为可以用[setCompletionRole()]、[setCompletionColumn()]和[setCaseSensitivity()]改变。

If the model is sorted on the column and role that are used for completion, 
