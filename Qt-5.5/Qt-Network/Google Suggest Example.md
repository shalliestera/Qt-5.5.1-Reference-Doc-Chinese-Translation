# Google Suggest Example `google 搜索建议范例` #
Qt 5.5 > Qt Network > Google Suggest Example

## Contents `目录`##
 * [GSuggestCompletion Class Declaration](#1.1)
 * [GSuggestCompletion Class Implementation](#1.2)
 * [SearchBox Class Declaration](#1.3)
 * [SearchBox Class Implementation](#1.4)

The example uses the `QNetworkAccessManager` to obtain the list of search recommendations by Google as the user types into a QLineEdit.
![/qtnetwork/images/googlesuggest-example.png](/qtnetwork/images/googlesuggest-example.png "")
qthelp://org.qt-project.qtnetwork.551/qtnetwork/images/googlesuggest-example.png  
The application makes use of the get function in QNetworkAccessManager to post a request and obtain the result of the search query sent to the Google search engine. The results returned are listed as clickable links appearing below the search box as a drop-down menu.

The widget is built up by a QLineEdit as the search box, and a QTreeView used as a popup menu below the search box.

## GSuggestCompletion Class Declaration ##

## GSuggestCompletion Class Implementation ##

## SearchBox Class Declaration ##

## SearchBox Class Implementation ##

