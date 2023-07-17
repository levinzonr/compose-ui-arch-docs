---
comments: true
---

# Actions
The `Actions` represen the User Interactions that are happening in our `Screen` and being handled by `Coordinator`. Not every interaction though should be considered an *Action*.


Here are few rules that higlight the purpose of `Actions` and their role in our Architecture

## Actions are triggered By User
The `Actions` are user interactions and can't be triggered without their involvement. Lets take this screen as an example. Here we would like to notify our parent component that the screen has been opened

```kotlin
data class Actions(
    val onTutorialIsCompleted: () -> Unit
)

@Composable
fun Screen(actions: Actions) {
    LaunchedEffect(Unit) {
        action.onScreenOpened()
    }
}

```
 While user has probably triggered the change in Navigation, first time rendering of Compose Screen is not User Interaction. thus it should not be part of the `Actions`. Whatever we wanted to do when user opens the screen, we can do it in `Route` or `Screen`.


The most only thing that is actually triggered by the User are UI components: `Button`, `TextField` etc.


```kotlin
data class LoginActions(
    val onLoginButtonClicked: () -> Unit,
    val onEmailChange: (String) -> Unit,
    val onPasswordChange: (String) -> Unit
)

@Composable
fun LoginScreen(val actions: LoginActions) {
    Column {
        TextField(onValueChange = actions.onEmailChange)
        TextField(onValueChange = actions.onPasswordChange)
        Button(onClick = actions.onLoginButtonClicked)
    }
}

```

## Actions drive the change in application state
This rule eliminates User Interactions that don't influence our Application State. The `Action` intention is to change our Application State, for example by changing the `State` of our `ViewModel`, navigate to another `Route`, things like that.

This really depends on the requirements you have for your `Screen` and what do you consider a `State`. For instance, lets say we have a component inside our screen that has a drop-down element.

```kotlin
fun SomeComponent(isRevealed: Boolean) {
    Column {
        Row {
            Text("Click to reveal")
            DropdownButton()
        }
    }
    if (isRevealed) {
        Text("More info")
    }
}
```

Lets say the `isRevealed` flag is not part of the Screen state, but rather an internal state of our component. In this case there is no point to expose the onClick action of our Dropdown button since we can this small component handle its own state


Some components are better left handling its own state as long as it the only state it manages and no-one esle has access to it, so that we stll has one source of truth.