# Beginners's Tutorial - Part 2
This second part follow the

# Covered features
PresenterWidget, Gatekeeper, PlaceManager and RestDispatch.

# Prerequisites
[Beginner's Tutorial - Part 1](?)

# Application Structure

```
├── application
│   ├── ApplicationModule.java
│   ├── ApplicationPresenter.java
│   ├── ApplicationView.java
│   ├── ApplicationView.ui.xml
│   ├── SuperDevModeUncaughtExceptionHandler.java
│   ├── home
│   │   ├── HomeModule.java
│   │   ├── HomePresenter.java
│   │   ├── HomeUiHandlers.java
│   │   ├── HomeView.java
│   │   ├── HomeView.ui.xml
│   │   └── widget
│   │       ├── SearchWidgetModule.java
│   │       ├── SearchWidgetPresenter.java
│   │       ├── SearchWidgetUiHandlers.java
│   │       ├── SearchWidgetView.java
│   │       └── SearchWidgetView.ui.xml
│   └── login
│       ├── LoginModule.java
│       ├── LoginPresenter.java
│       ├── LoginUiHandlers.java
│       ├── LoginView.java
│       └── LoginView.ui.xml
├── gin
│   └── ClientModule.java
└── place
    └── NameTokens.java

```

1. LoginPresenter covers Gatekeeper
1. HomePresenter covers RestDispatch
1. SearchPresenterWidget covers PresenterWidget and NestedSlots

## PresenterWidget - Creating a search widget
A [PresenterWidget](?) allow you to reuse UI components throughout an application. It has the same functionality as a GWT Widget but implements the MVP pattern. Let's say we want to add a very basic "Contact" search functionality. If we were to reuse this functionality on multiple presenters, the best way to achieve this would be with a PresenterWidget.

Let's begin by creating a new package under `home/` and call it `widget`.

Here is the `SearchPresenterWidget`:

```java
public class SearchWidgetPresenter extends PresenterWidget<SearchWidgetPresenter.MyView>
        implements SearchWidgetUiHandlers {
    public interface MyView extends View, HasUiHandlers<SearchWidgetUiHandlers> {
    }

    @Inject
    public SearchWidgetPresenter(
            EventBus eventBus,
            MyView view) {
        super(eventBus, view);

        getView().setUiHandlers(this);
    }

    @Override
    public void search(String searchTerm) {
        // Send request to the server
    }
}
```

And this is the `SearchWidgetView`:

```java
public class SearchWidgetView extends ViewWithUiHandlers<SearchWidgetUiHandlers>
        implements SearchWidgetPresenter.MyView {
    interface Binder extends UiBinder<HTMLPanel, SearchWidgetView> {
    }

    @UiField
    TextBox searchField;
    @UiField
    Button searchButton;

    @Inject
    public SearchWidgetView(Binder uiBinder) {
        initWidget(uiBinder.createAndBindUi(this));
    }

    @UiHandler("searchButton")
    void onSearch(ClickEvent clickEvent) {
        String searchFieldText = searchField.getText();

        if (!Strings.isNullOrEmpty(searchFieldText)) {
            getUiHandlers().search(searchFieldText);
        }
    }
}
```

We only need two fields for this View, a search box and a confirm button.

```xml
<ui:UiBinder xmlns:ui='urn:ui:com.google.gwt.uibinder'
             xmlns:g='urn:import:com.google.gwt.user.client.ui'>
    <g:HTMLPanel>
        <h3>Search</h3>
        <g:TextBox ui:field="searchField"/>
        <g:Button ui:field="searchButton"/>
    </g:HTMLPanel>
</ui:UiBinder>
```

```java
import com.gwtplatform.mvp.client.UiHandlers;

public interface SearchWidgetUiHandlers extends UiHandlers {
    void search(String searchTerm);
}
```

## Gatekeeper - Creating a login page
## PlaceManager - Navigating to another place
## RestDispatch - Sending requests to a server
