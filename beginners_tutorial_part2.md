# GWTP Beginner’s Tutorial: Toaster Launcher Part 2
In the previous article, you saw how to create a Presenter and its associated view, you also say how to use UiHandlers to delegate some of the View events to the Presenter and finally how to use UiBinder to declare DOM elements with XML markup and access it from the View.

In this part, we'll continue to build what we need for our toaster's incredible journey into space.

This tutorial will go over the following features:
* [Gatekeeper: Protecting your assets](#Gatekeeper: Protecting your assets)
* PlaceManager: Going from one place to another
* PresenterWidget: Reusable controls with their own logic
* RestDispatch: Communicating the RESTful way

This is the new application structure:

```
├── ApplicationModule.java
├── ApplicationPresenter.java
├── ApplicationView.java
├── ApplicationView.ui.xml
├── CurrentUser.java
├── LoggedInGatekeeper.java
├── launcher
│   ├── LauncherModule.java
│   ├── LauncherPresenter.java
│   ├── LauncherUiHandlers.java
│   ├── LauncherView.java
│   ├── LauncherView.ui.xml
│   └── widget
│       ├── ToasterPresenterWidget.java
│       ├── ToasterViewWidget.java
│       ├── ToasterViewWidget.ui.xml
│       └── ToasterWidgetUiHandlers.java
└── login
    ├── LoginModule.java
    ├── LoginPresenter.java
    ├── LoginUiHandlers.java
    ├── LoginView.java
    └── LoginView.ui.xml
```

## Gatekeeper: Protecting your assets
A [Gatekeeper][Gatekeeper] is what you use when you want to protect a Place (a Presenter having a ProxyPlace) from unauthorized access. If your application has a login page or an admin section, you want a Gatekeeper. When a Gatekeeper is used on a Presenter, it will prevent revealing the Presenter until x condition is met. x condition will be verified on the Gatekeeper's `canReveal()` method. If it returns true, the Presenter will be revealed if not, well, it will not...

This is the Gatekeeper we're going to use:

```java
import com.google.inject.Inject;
import com.gwtplatform.mvp.client.annotations.DefaultGatekeeper;
import com.gwtplatform.mvp.client.proxy.Gatekeeper;

@DefaultGatekeeper
public class LoggedInGatekeeper implements Gatekeeper {
    private CurrentUser currentUser;

    @Inject
    public LoggedInGatekeeper(CurrentUser currentUser) {
        this.currentUser = currentUser;
    }

    @Override
    public boolean canReveal() {
        return currentUser.isLoggedIn();
    }
}
```

The `@DefaultGatekeeper` annotation will tell GWTP to use this Gatekeeper on every Presenter having a `ProxyPlace`. But what if you don't want to use the default Gatekeeper on a specific Presenter? Well, there's the `@NoGatekeeper` annotation for that. We're going to use it in the next example.

In the application context, we're going to use a Gatekeeper to prevent anyone with unauthorized access from launching the toaster. By using what we previously saw in the first part tutorial, we're going to create a LoginPresenter:

```java
public class LoginPresenter extends Presenter<LoginPresenter.MyView, LoginPresenter.MyProxy>
        implements LoginUiHandlers {
    @ProxyStandard
    @NameToken(NameTokens.LOGIN)
    @NoGatekeeper
    interface MyProxy extends ProxyPlace<LoginPresenter> {
    }

    interface MyView extends View, HasUiHandlers<LoginUiHandlers> {
    }

    @Inject
    LoginPresenter(
            EventBus eventBus,
            MyView view,
            MyProxy proxy) {
        super(eventBus, view, proxy);

        getView().setUiHandlers(this);
    }
}
```

Now you can see that we've used `@NoGatekeeper` on `MyProxy` interface. We did this because we don't want `LoggedInGatekeeper` to apply to this Place as it's the first page the user will land on. Then, we're going to create the `LoginView`. Personally, I prefer to create the XML markup file first:

This is `LoginView.ui.xml`:

```xml
<ui:UiBinder xmlns:ui='urn:ui:com.google.gwt.uibinder'
             xmlns:g='urn:import:com.google.gwt.user.client.ui'>
    <g:HTMLPanel>
        <g:TextBox ui:field="username"/><br/>
        <g:PasswordTextBox ui:field="password"/><br/>
        <g:Button ui:field="confirm" text="Login"/>
    </g:HTMLPanel>
</ui:UiBinder>
```

Then `LoginView.java`:

```java
import javax.inject.Inject;

import com.google.gwt.event.dom.client.ClickEvent;
import com.google.gwt.uibinder.client.UiBinder;
import com.google.gwt.uibinder.client.UiField;
import com.google.gwt.uibinder.client.UiHandler;
import com.google.gwt.user.client.ui.Button;
import com.google.gwt.user.client.ui.PasswordTextBox;
import com.google.gwt.user.client.ui.TextBox;
import com.google.gwt.user.client.ui.Widget;
import com.gwtplatform.mvp.client.ViewWithUiHandlers;

public class LoginView extends ViewWithUiHandlers<LoginUiHandlers> implements LoginPresenter.MyView {
    interface Binder extends UiBinder<Widget, LoginView> {
    }

    @UiField
    Button confirm;
    @UiField
    TextBox username;
    @UiField
    PasswordTextBox password;

    @Inject
    LoginView(
            Binder uiBinder) {
        initWidget(uiBinder.createAndBindUi(this));
    }

    @UiHandler("confirm")
    void onConfirm(ClickEvent event) {
        // TODO: Confirm that the user and password matches
    }
}
```

And don't forget to create the UiHandler:

```java
import com.gwtplatform.mvp.client.UiHandlers;

public interface LoginUiHandlers extends UiHandlers {
}
```

So, we now have a LoginPresenter, a LoginView and a LoginUiHandlers, we're going to add a bit of logic to the Presenter so that a user can log in. Because this is a tutorial and showing you how to login via a server is out of context, we're going to put the credentials in the LoginPresenter. However, I **strongly discourage** you to store any user credentials on the client-side as it's not a recommended practice from a security perspective. Moving on...

We're going to add the secret credentials to operate our toaster into the LoginPresenter:

```java
private static final String USERNAME = "admin";
private static final String PASSWORD = "admin123";
```

And then, we need to add a `confirm()` method to the UiHandler:

```java
import com.gwtplatform.mvp.client.UiHandlers;

public interface LoginUiHandlers extends UiHandlers {
    void confirm(String username, String password);
}
```

And then override the method in the LoginPresenter:

```java
@Override
public void confirm(String username, String password) {
    if (validateCredentials(username, password)) {
        currentUser.setLoggedIn(true);

        // TODO: Navigate to another place.
    }
}

private boolean validateCredentials(String username, String password) {
    return username.equals(USERNAME) && password.equals(PASSWORD);
}
```

And then from the LoginView we call the `confirm()` method.

```java
@UiHandler("confirm")
void onConfirm(ClickEvent event) {
    getUiHandlers().confirm(username.getText(), password.getText());
}
```

## PlaceManager: Going from one place to another
The [PlaceManager][PlaceManager] makes the link between GWT history API and GWTP ProxyPlaceAbstract. At high level, it takes care of revealing Places in a pretty straight forward way.

We're going to use the PlaceManager to navigate to the LauncherPresenter. In order to do this, we need to inject `PlaceManager` into the LoginPresenter:

```java
private PlaceManager placeManager;

@Inject
LoginPresenter(
        EventBus eventBus,
        MyView view,
        MyProxy proxy,
        PlaceManager placeManager) {
    super(eventBus, view, proxy, ApplicationPresenter.SLOT_MAIN);

    this.placeManager = placeManager;
    getView().setUiHandlers(this);
}
```

Then all we need is a `PlaceRequest` that we're going to create using the `PlaceRequest.Builder()`:

```java
@Override
public void confirm(String username, String password) {
    if (validateCredentials(username, password)) {
        currentUser.setLoggedIn(true);

        PlaceRequest placeRequest = new PlaceRequest.Builder()
                .nameToken(NameTokens.HOME)
                .build();
        placeManager.revealPlace(placeRequest);
    }
}
```

That's it! Now when the user enter its credentials, the LoginPresenter will validate them and set `CurrentUser.isLoggedIn()` to `true`. The LauncherPresenter will then be revealed by the PlaceManager after `LoggedInGatekeeper.canReveal()` is called.

## PresenterWidget: Reusable controls with their own logic
It often happens in an application that you would like to reuse UI components in more than one place. For example: a search bar, a contact form or a menu. These components are not navigable but it is possible to use them the same way as you would use Presenters. Using [PresenterWidgets][PresenterWidgets], your UI components can be decoupled into a Presenter and a View. You can then inject it into any other Presenters, as many times as you like­, as long as you have a slot in which to set it.

For our application, we're going to create a PresenterWidget to communicate with our toaster when it's out there into the infinite of space.

This is the `ToasterPresenterWidget.java`:

```java
import com.google.inject.Inject;
import com.google.web.bindery.event.shared.EventBus;
import com.gwtplatform.mvp.client.HasUiHandlers;
import com.gwtplatform.mvp.client.PresenterWidget;
import com.gwtplatform.mvp.client.View;

public class ToasterPresenterWidget extends PresenterWidget<ToasterPresenterWidget.MyView>
        implements ToasterWidgetUiHandlers {
    interface MyView extends View, HasUiHandlers<ToasterWidgetUiHandlers> {
    }

    @Inject
    public ToasterPresenterWidget(
            EventBus eventBus,
            MyView view) {
        super(eventBus, view);

        getView().setUiHandlers(this);
    }
}
```

As you can see, ToasterPresenterWidget extends `PresenterWidget` rather that `Presenter`. It also doesn't have the `MyProxy` interface you'd declare in a regular Presenter.

This is the `ToasterViewVidget.ui.xml`:

```java
<ui:UiBinder xmlns:ui='urn:ui:com.google.gwt.uibinder'
             xmlns:g='urn:import:com.google.gwt.user.client.ui'>
    <g:HTMLPanel>
        <g:Button ui:field="fetchButton"/>
        <g:TextBox ui:field="name"/>
        <g:TextBox ui:field="coordinates"/>
        <g:TextBox ui:field="thrust"/>
    </g:HTMLPanel>
</ui:UiBinder>
```

This is the `ToasterViewWidget.java`:

```java
import com.google.gwt.event.dom.client.ClickEvent;
import com.google.gwt.uibinder.client.UiBinder;
import com.google.gwt.uibinder.client.UiHandler;
import com.google.gwt.user.client.ui.HTMLPanel;
import com.google.inject.Inject;
import com.gwtplatform.mvp.client.ViewWithUiHandlers;

public class ToasterViewWidget extends ViewWithUiHandlers<ToasterWidgetUiHandlers>
        implements ToasterPresenterWidget.MyView {
    interface Binder extends UiBinder<HTMLPanel, ToasterViewWidget> {
    }

    @UiField
    Button fetchButton;
    @UiField
    TextBox name;
    @UiField
    TextBox coordinates;
    @UiField
    TextBox thrust;

    @Inject
    public ToasterViewWidget(Binder binder) {
        initWidget(binder.createAndBindUi(this));
    }

    @UiHandler("fetchButton")
    void onFetchToaster(ClickEvent clickEvent) {
        // TODO: Fetch toaster data from server
    }
}
```

This is the `ToasterWidgetUiHandlers.java`:

```java
import com.gwtplatform.mvp.client.UiHandlers;

public interface ToasterWidgetUiHandlers extends UiHandlers {
    void fetchToaster();
}
```

Now we're going to override the method into the ToasterPresenterWidget:

```java
@Override
public void fetchToaster() {
    // TODO: Send request to server
}
```

Now to set this PresenterWidget into its parent Presenter, we need to inject it and set it in a [Slot][Slot]. Like this:

```java
public static final Slot SLOT_CONTENT = new Slot();

private final ToasterPresenterWidget toasterPresenterWidget;

@Inject
LauncherPresenter(
        EventBus eventBus,
        MyView view,
        MyProxy proxy,
        ToasterPresenterWidget toasterPresenterWidget) {
    super(eventBus, view, proxy, ApplicationPresenter.SLOT_MAIN);

    this.toasterPresenterWidget = toasterPresenterWidget;

    getView().setUiHandlers(this);
    setInSlot(SLOT_CONTENT, toasterPresenterWidget);
}
```

The `setInSlot()` method allow us to set the PresenterWidget we've injected into the Slot you just declared.

## RestDispatch: Communicating the RESTful way
REST is everywhere nowadays and GWTP has a client library to communicate with a server in a RESTful way. [RestDispatch][RestDispatch] can be used independently of GWTP which makes it great for any GWT application. Everything you send over the wire must be serializable in JSON.

[:Gatekeeper](http://dev.arcbees.com/gwtp/core/security/)
[:PlaceManager](http://dev.arcbees.com/gwtp/core/navigation/reveal-places.html)
[:PresenterWidgets](http://dev.arcbees.com/gwtp/core/presenters/index.html)
[:Slot](http://dev.arcbees.com/gwtp/core/slots/)
[:RestDispatch](http://dev.arcbees.com/gwtp/communication/)
