---
comments: true
---


# Screen
The `Screen` is the visual representation of the feature that we are working on. It is  component that renders our UI based on the given `State` and emits `Actions` for our `Coordinator` to handle. Here are a few rules we would like our `Screen` to flow.

- **Easily Previewable** - We should be able easily preview our screen with different states, across different device sizes and modes, light or dark
- **Consise** - The `Screen` component should be relatevely short, easy to read and understand. Encapsulating parts of it UI into separate components is a must. Also be aware of the number of nested blocks, having it small as possible makes the code much more readable
- **State less**. The `Screen` should consume state but not manage this state. It doesn't allowed to reference any other component is not UI only. 
  


## State consumption
The UI of the Screen is Rendered based on the Provided state, it can a ViewModel state or it can be a composable component state. It is good practice to let you component know a little as possible. Since the `Screen` is split into UI components we can make these components rely only the part of the state that is provided to the `Screen`. So the deeper we go through the UI tree the less components know about our specific feature and its state

This makes these components more re-usable, agnostic to a specific ViewModel state, and causes less re-rendering wen recomposition

```kotlin

data class ExampleState(
    val user: User,
    val products: List<Products>
)

data class ExampleActions(
    val onUserClicked: () -> Unit,
    val onProductClick: (Product) -> Unit
)

@Composable
fun ExampleScreen(
    state: ExampleState,
    actions: ExampleActions
) {
    LazyColumn {
        item {
            UserItem(state.user, actions.onUserClicked)
        }

        items(state.products) { product -> 
            ProductItem(product, actions.onProductClicked)
        }
    }
}

@Composable
fun UserItem(user: User, onClick: () -> Unit) { /* */ }
@Composable

fun ProductItem(user: Product, onClick: (Product) -> Unit) { /* */ }

```


## Effects
The Compose Side Effects can also be a part of screen or a specific component. However, it is important to be aware the goal of that effects, what they are trying to achieve.


Lets look a at following screen. We have Pager State and just received a new requirement to track every time user changes the page.

```kotlin

data class ScreenActions(val onPageChange: (Int) -> Unit)

@Composable
fun Screen(
    val pagerState: PagerState
     actions: ScreenActions
) {
    LaunchedEffect(pagerState) {
        snapshotFlow { pagerState.currentPage }
            .onEach(actions::onPageChange)
            .launchIn(this)
    }

    ItemsPager(state = pagerState)
}

class Coordinator {
    
    fun handlePageChange(page: Int) {
        // our logic
    }
}
```

So lets break down what we have here, once user has changed the page in the `Pager`
1. Our flow in `LaunchedEffect` detects the change and gets triggered
2. New page number is then being passed with `Actions`
3. The `Action` then is handled by our `Route` or `Coordinator`

So while this actually doing what we wanted it to do, the amount interactions and component tied to this actions is a bit redundat. Since the `PagerState` is external: we don't control the state, we only get notified about the changes. And it actually does nothing to the `Screen` we are dealing with. All that raises the question, why would we have this effect as part of the `Screen`

To fix this we can just move this effect into the `Coordinator`. This makes both `Screen` and `Actions` more consise and removes redundant Logic form them.

```kotlin
class Coordinator(val pagerState: PagerState, scope: CoroutineScope) {
    init {
        snapshotFlow { pagerState.currentPage }
            .onEach { /* our track loginc */ }
            .launchIn(scope)
    }
}
