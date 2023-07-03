# State
The state is the bread and butter of Compose. This page describes the role of State in our architecture, different types of states and should and should not be a part of it


> This page will mostly rely on features and components familiar to android developers (like androidx.ViewModel) but the same principles can applied to any other components and tech stack powered by kotlin

## ViewModel State
In android, our ViewModel acts a mediator between the Business rules of our application and its presentation logic. The name of this component might change depending on your technical stack but not its responsibility. 

The State of a ViewModel is a representation of both our UI and business rules. In kotlin this State can be presented in many ways, by far the Most popular are `data class` and `sealed interface` or `sealed class`.

It is recommended to have one State per ViewModel as it make

### Data class
The data class based is pretty straightforward and can be used in situations when you `Screen` has many things displayed at once.

When combined with `StateFlow` we can easy and safely mutate it inside our ViewModel, while making sure only when state gets updated and changed, it will be consumed by our UI

Lets take a look at this classic example of ViewModel state

```kotlin
data class ExampleState(
    val refreshing: Boolean = false,
    val items: List<Item> = emptyList(),
    val error: Error? = null
)
```

In our `ViewModel` this state can be mutated by using `update` extension function

```kotlin
class ExampleViewModel(
    private val getItemsUseCase: GetItemsUseCase
) : ViewModel() {

    private val _stateFlow = MutableStateFlow(ExampleState())

    // .asStateFlow() make our mutable flow immutable
    // making our ViewModel the only place where we can do
    // state updates
    val stateFlow = _stateFlow().asStateFlow()

    init {
        viewModelScope.launch {
            _stateFlow.update { state -> 
                state.copy(refreshing = true)
            }
            // do some background work to load items
            val result: Result<Items> = getItemsUseCase()

            _stateFlow.update { state -> 
                state.copy(
                    refreshing = false,
                    items = result.getOrEmpty(),
                    error = result.exceptionOrNull()
                )
            }

        }
    }
}
```

### Sealed Interface / Sealed Class
Describing your state using can `sealed interface` or `sealed class` provide some benefits comparing to `data class`. For one, it can much more clearly describe whats going one on the Screen as your state basically becomes a distinct set of possible outcomes, where one fuses into another, creating a small state machine

```kotlin
sealed interface PaymentState {
    data class Summary(val price: Price): PaymentState
    object Processing: PaymentState
    object Completed: PaymentState
}
```

The same thing can be achieved with data class, but this would require to introduce three booleans which introduces *2^3=8* possible states instead of desired three, making it much easier to make a mistake. 

> And, of course these two can be combined, you can have `sealed interface` be a a part of bigger `data class` and different `data class` can represent `sealed interface` nodes.

And our ViewModel can then look like this 

```kotlin
class PaymentViewModel() {

    init {
        _stateFlow.update { PaymentState.Summary(getPrice())) }
    }

    fun pay() {
        viewModelScope.launch {
            _stateFlow.update { PaymentState.Processing }
            // background stuff
            _stateFlow.update { PaymentState.Completed }
        }
    }
}
```


## Composable State
Another type of state that can be exposed to our Screen is a State of other composable components. Pagers, Lazy Lists, state of the Bottom Sheets and scaffolds. 
