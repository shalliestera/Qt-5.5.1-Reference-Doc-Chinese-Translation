# Google Suggest Example `google 搜索建议范例` #
Qt 5.5 > Qt Network > Google Suggest Example

## Contents `目录`##
 * [GSuggestCompletion Class Declaration](#1.1) GSuggestCompletion 类声明
 * [GSuggestCompletion Class Implementation](#) GSuggestCompletion 类实现
 * [SearchBox Class Declaration](#) SearchBox 类声明
 * [SearchBox Class Implementation](#) SearchBox 类实现

The example uses the [QNetworkAccessManager]() to obtain the list of search recommendations by Google as the user types into a QLineEdit.

这个范例使用 [QNetworkAccessManager]() 在用户键入一个 QLineEdit 时获取谷歌搜索建议列表。

![./images/googlesuggest-example.png](./qtnetwork/images/googlesuggest-example.png)

The application makes use of the `get` function in QNetworkAccessManager to post a request and obtain the result of the search query sent to the Google search engine. The results returned are listed as clickable links appearing below the search box as a drop-down menu.

这个应用程序利用 QNetworkAccessManager 中的 `get` 函数来 post 一个请求，并获取发送到谷歌搜索引擎的搜索请求的结果。返回的结果作为一个可点击的下拉列表的形式显示在 search box 下方。

The widget is built up by a QLineEdit as the search box, and a QTreeView used as a popup menu below the search box.

这个 widget 由一个作为 search box 的QLineEdit，和一个作为 search box 下方的弹出菜单的 QTreeView组成。

## GSuggestCompletion Class Declaration ##
<span id="1.1"></span>
这个类实现一个事件过滤器和若干个函数来显示搜索结果及决定何时、如何执行搜索。
```c++
class GSuggestCompletion : public QObject
{
    Q_OBJECT

public:
    GSuggestCompletion(QLineEdit *parent = 0);
    ~GSuggestCompletion();
    bool eventFilter(QObject *obj, QEvent *ev) Q_DECL_OVERRIDE;
    void showCompletion(const QStringList &choices);

public slots:

    void doneCompletion();
    void preventSuggest();
    void autoSuggest();
    void handleNetworkData(QNetworkReply *networkReply);

private:
    QLineEdit *editor;
    QTreeWidget *popup;
    QTimer *timer;
    QNetworkAccessManager networkManager;
};
```
这个类连接到一个 QLineEdit ，并用一个 QTreeWidget 来显示结果。一个 QTimer 控制由 QNetworkAccessManager 执行的网络请求的开始。

## GSuggestCompletion Class Implementation ##

我们从定义一个包含用于`谷歌查询`的URL的常量开始。这是查询的基础。键入 search box 的字符会把自身追加到 请求中以执行搜索。

```c++
#include "googlesuggest.h"
#define GSUGGEST_URL "http://google.com/complete/search?output=toolbar&q=%1"
```

在构造函数中，我们把这个 QSugguestCompetion 实例的父元素设置为传入的 QLineEdit。为简单起见，QLineEdit 也储存于显式的 `editor` 成员变量中。

我们随后创建一个 QTreeWidget 作为顶层 widget，并配置各种属性给予它弹出式 widget 的外观。
这个 widget 由谷歌搜索建议API请求的结果填充。

此外，我们在 QTreeWidget 上安装一个 GSuggestCompletion 实例作为一个事件过滤器，
并连接 `itemClicked()` 信号和 `doneCompletion()` 槽。

当用户停止输入 500 毫秒后，一个单发的 [QTimer]() 会被用来开始请求。

最后，我们连接 networkManager 的 `finished()` 信号和 `handleNetworkData` 槽来处理输入的数据。

```c++
GSuggestCompletion::GSuggestCompletion(QLineEdit *parent): QObject(parent), editor(parent)
{
    popup = new QTreeWidget;
    popup->setWindowFlags(Qt::Popup);
    popup->setFocusPolicy(Qt::NoFocus);
    popup->setFocusProxy(parent);
    popup->setMouseTracking(true);

    popup->setColumnCount(1);
    popup->setUniformRowHeights(true);
    popup->setRootIsDecorated(false);
    popup->setEditTriggers(QTreeWidget::NoEditTriggers);
    popup->setSelectionBehavior(QTreeWidget::SelectRows);
    popup->setFrameStyle(QFrame::Box | QFrame::Plain);
    popup->setHorizontalScrollBarPolicy(Qt::ScrollBarAlwaysOff);
    popup->header()->hide();

    popup->installEventFilter(this);

    connect(popup, SIGNAL(itemClicked(QTreeWidgetItem*,int)),
            SLOT(doneCompletion()));

    timer = new QTimer(this);
    timer->setSingleShot(true);
    timer->setInterval(500);
    connect(timer, SIGNAL(timeout()), SLOT(autoSuggest()));
    connect(editor, SIGNAL(textEdited(QString)), timer, SLOT(start()));

    connect(&networkManager, SIGNAL(finished(QNetworkReply*)),
            this, SLOT(handleNetworkData(QNetworkReply*)));
}
```

由于 QTreeWidget 弹出菜单被实例化为顶层 widget ，析构函数必须显式地删除它以避免内存泄露。

```c++
GSuggestCompletion::~GSuggestCompletion()
{
    delete popup;
}
```

事件过滤器处理交付给弹出菜单的 mouse press 和 key press 事件。对于 mouse press events，
我们只需隐藏弹出菜单并把焦点返回给 editor widget，然后返回 true 以阻止进一步的事件处理。

Key 事件处理被实现以便 Enter 和 Return 执行选中的链接，而 Escape 键隐藏弹出菜单。
由于我们想能够使用不同的导航键导航建议列表，我们从 eventFilter 返回 false ，让 Qt 继续常规事件处理。

对于其他按键，事件将被传递给 editor widget，弹出菜单隐藏。
这种方式下用户的输入不会被 completion list 的弹出打断。

```
bool GSuggestCompletion::eventFilter(QObject *obj, QEvent *ev)
{
    if (obj != popup)
        return false;

    if (ev->type() == QEvent::MouseButtonPress) {
        popup->hide();
        editor->setFocus();
        return true;
    }

    if (ev->type() == QEvent::KeyPress) {

        bool consumed = false;
        int key = static_cast<QKeyEvent*>(ev)->key();
        switch (key) {
        case Qt::Key_Enter:
        case Qt::Key_Return:
            doneCompletion();
            consumed = true;

        case Qt::Key_Escape:
            editor->setFocus();
            popup->hide();
            consumed = true;

        case Qt::Key_Up:
        case Qt::Key_Down:
        case Qt::Key_Home:
        case Qt::Key_End:
        case Qt::Key_PageUp:
        case Qt::Key_PageDown:
            break;

        default:
            editor->setFocus();
            editor->event(ev);
            popup->hide();
            break;
        }

        return consumed;
    }

    return false;
}
```

showCompletion() 函数将查询返回的结果填充到 QTreeWidget 中。它采用一个建议搜索词的 [QStringList]()。

```c++
void GSuggestCompletion::showCompletion(const QStringList &choices)
{

    if (choices.isEmpty())
        return;

    const QPalette &pal = editor->palette();
    QColor color = pal.color(QPalette::Disabled, QPalette::WindowText);

    popup->setUpdatesEnabled(false);
    popup->clear();
    for (int i = 0; i < choices.count(); ++i) {
        QTreeWidgetItem * item;
        item = new QTreeWidgetItem(popup);
        item->setText(0, choices[i]);
        item->setTextColor(0, color);
    }
    popup->setCurrentItem(popup->topLevelItem(0));
    popup->resizeColumnToContents(0);
    popup->setUpdatesEnabled(true);

    popup->move(editor->mapToGlobal(QPoint(0, editor->height())));
    popup->setFocus();
    popup->show();
}
```

由列表中的每一项创建的 QTreeWidgetItem 被插入到 QTreeWidget 。
最后，调整弹出菜单的位置和尺寸，以确保它出现在 editor 下正确的位置，然后显示出来。

Enter 或 Return 键按下时调用的 doneCompletion() 函数会停止 timer 以阻止更多请求，
并传递选中项的文本给 editor 。使 `editor` QLineEdit 发出 returnPressed() 信号，
用于程序关联，打开不同的网页。

```c++
void GSuggestCompletion::doneCompletion()
{
    timer->stop();
    popup->hide();
    editor->setFocus();
    QTreeWidgetItem *item = popup->currentItem();
    if (item) {
        editor->setText(item->text(0));
        QMetaObject::invokeMethod(editor, "returnPressed");
    }
}
```

当定时器超时，`autoSuggest()` 槽就被调用，并使用 editor 中的文本来建立完整搜索请求。
随后这个请求被传递给 [QNetworkAccessManager]() 的 `get()` 函数来开始请求。

```c++
void GSuggestCompletion::autoSuggest()
{
    QString str = editor->text();
    QString url = QString(GSUGGEST_URL).arg(str);
    networkManager.get(QNetworkRequest(QString(url)));
}
```

`preventSuggest()`函数停止定时器以阻止更多请求。

```c++
void GSuggestCompletion::preventSuggest()
{
    timer->stop();
}
```

当网络请求结束， QNetworkAccessManager 通过 networkReply 对象传递从服务器收到的数据。

```c++
void GSuggestCompletion::handleNetworkData(QNetworkReply *networkReply)
{
    QUrl url = networkReply->url();
    if (!networkReply->error()) {
        QStringList choices;

        QByteArray response(networkReply->readAll());
        QXmlStreamReader xml(response);
        while (!xml.atEnd()) {
            xml.readNext();
            if (xml.tokenType() == QXmlStreamReader::StartElement)
                if (xml.name() == "suggestion") {
                    QStringRef str = xml.attributes().value("data");
                    choices << str.toString();
                }
        }

        showCompletion(choices);
    }

    networkReply->deleteLater();
}
```

为了从回复中提取数据，使用从 [QIODevice]() 继承的 `readAll()` 函数，它返回一个 [QByteArray]。
因为数据是用 XML 编码的，于是用 [QXmlStreamReader] 来遍历数据，并把搜索结果提取为 QStrings，
将它们汇入用来填充弹出菜单的两个 QStringList。

最后，使用 `deleteLater` 函数安排 [QNetworkReply] 对象的删除。

## SearchBox Class Declaration ##

SearchBox 类继承自 QLineEdit ，并添加了保护槽 `doSearch()`。

一个 `GSuggestCompletion` 成员为 SearchBox 提供请求功能和从谷歌搜索引擎返回的建议。

```c++
#include <QLineEdit>

class GSuggestCompletion;

class SearchBox: public QLineEdit
{
    Q_OBJECT

public:
    SearchBox(QWidget *parent = 0);

protected slots:
    void doSearch();

private:
    GSuggestCompletion *completer;
```

## SearchBox Class Implementation ##

search box 的构造函数实例化 GSuggestCompletion 对象，并关联 returnPressed() 信号 到 doSearch() 槽。

```c++
SearchBox::SearchBox(QWidget *parent): QLineEdit(parent)
{
    completer = new GSuggestCompletion(this);

    connect(this, SIGNAL(returnPressed()),this, SLOT(doSearch()));

    setWindowTitle("Search with Google");

    adjustSize();
    resize(400, height());
    setFocus();
}
```

`doSearch()` 函数阻止 completer 向搜索引擎发送更多查询请求。

而且，这个函数提取出选中的搜索短语，并用 [QDesktopServices]() 在默认浏览器中打开。

```c++
void SearchBox::doSearch()
{
    completer->preventSuggest();
    QString url = QString(GSEARCH_URL).arg(text());
    QDesktopServices::openUrl(QUrl(url));
}
```

文件
* [googlesuggest/googlesuggest.cpp]()
* [googlesuggest/googlesuggest.h]()
* [googlesuggest/searchbox.cpp]()
* [googlesuggest/searchbox.h]()
* [googlesuggest/main.cpp]()
* [googlesuggest/googlesuggest.pro]()
