# flutter bloc vs redux


Now that I know how to the write redux in Flutter, I think it's pretty great.

That said, it took a long time to understand how to take the examples listed on the flutter redux docs and actually make it look like redux. Documentation can go a long way for redux in the flutter community. I believe it's largely ignored because the community rallies around Provider or Bloc, which are great! I'm unsure about the boilerplate argument at scale with Bloc, but even if there is slightly more boilerplate I don't think it's enough to warrants the context switch between the patterns. If you're comfortable with redux already, I would say go for it. My friend at work already knew how to read my code coming fom a react native and even a Vue redux implimentation.

For example, this is a "CounterBloc" example Bloc gives:
```dart
enum CounterEvent { increment, decrement }

class CounterBloc extends Bloc<CounterEvent, int> {
  @override
  int get initialState => 0;

  @override
  Stream<int> mapEventToState(CounterEvent event) async* {
    switch (event) {
      case CounterEvent.decrement:
        yield state - 1;
        break;
      case CounterEvent.increment:
        yield state + 1;
        break;
    }
  }
}
```

Here's the same "Reducer" and initial state written in flutter redux


```dart
class Decrement {}

class Increment {}

int countReducer([int state = 0, dynamic action]) {
  switch (action.runtimeType) {
    case Increment:
      return state + 1; // or action.amount to increase by arbitrary amounts
    case Decrement:
      return state - 1;
  }
}
```

I generally don't like this example, because state is almost never just one value but lets just go with it

Here's how Bloc connects it to the view

```dart
class CounterPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final CounterBloc counterBloc = context.bloc<CounterBloc>();

    return Scaffold(
      appBar: AppBar(title: Text('Counter')),
      body: BlocBuilder<CounterBloc, int>(
        builder: (context, count) {
          return Center(
            child: Text(
              '$count',
              style: TextStyle(fontSize: 24.0),
            ),
          );
        },
      ),
      floatingActionButton: Column(
        crossAxisAlignment: CrossAxisAlignment.end,
        mainAxisAlignment: MainAxisAlignment.end,
        children: <Widget>[
          Padding(
            padding: EdgeInsets.symmetric(vertical: 5.0),
            child: FloatingActionButton(
              child: Icon(Icons.add),
              onPressed: () {
                counterBloc.add(CounterEvent.increment);
              },
            ),
          ),
          Padding(
            padding: EdgeInsets.symmetric(vertical: 5.0),
            child: FloatingActionButton(
              child: Icon(Icons.remove),
              onPressed: () {
                counterBloc.add(CounterEvent.decrement);
              },
            ),
          ),
        ],
      ),
    );
  }
}
```

Here's how I would do it in flutter redux (assuming you have the global AppState from Syphon lol)

```dart
class CounterPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) => StoreConnector<AppState, Props>(
      distinct: true,
      converter: (Store<AppState> store) => Props.mapStateToProps(store),
      builder: (context, props) {
        return Scaffold(
          appBar: AppBar(title: Text('Counter')),
          body: Center(
            child: Text(
              props.count,
              style: TextStyle(fontSize: 24.0),
            ),
          ),
          floatingActionButton: Column(
            crossAxisAlignment: CrossAxisAlignment.end,
            mainAxisAlignment: MainAxisAlignment.end,
            children: <Widget>[
              Padding(
                padding: EdgeInsets.symmetric(vertical: 5.0),
                child: FloatingActionButton(
                  child: Icon(Icons.add),
                  onPressed: () => props.onIncrement(),
                ),
              ),
              Padding(
                padding: EdgeInsets.symmetric(vertical: 5.0),
                child: FloatingActionButton(
                  child: Icon(Icons.remove),
                  onPressed: () => props.onDecrement(),
                ),
              ),
            ],
          ),
        );
      });
}

class Props extends Equatable {
  final int count;
  final Function onIncrement;
  final Function onDecrement;

  Props({
    @required this.count,
    @required this.onIncrement,
    @required this.onDecrement,
  });

  static Props mapStateToProps(Store<AppState> store) => Props(
        count: store.state.countStore,
        onIncrement: store.dispatch(Increment),
        onDecrement: store.dispatch(Decrement),
      );

  @override
  List<Object> get props => [
        count,
      ];
}
```

The part most people get hung up on is that view model at the end. While I understand boilerplate, that view model in-between the view and the state comes in handy when you want to pivot to using a selector on a state values, like I do with the outbox in Syphon.

``` dart
static _Props mapStateToProps(Store<AppState> store, String roomId) => _Props( 
      messages: latestMessages(
        wrapOutboxMessages(
          messages: store.state.roomStore.rooms[roomId].messages,
          outbox: store.state.roomStore.rooms[roomId].outbox,
        ),
      ),
)
```

This is part of the reason I wanted to create the app to begin with because the pattern is not as bad as people make it out to be in the community imo. This will likely bring a lot of hate toward me, but please go look at Syphon and see for yourself! I like all of the platforms, and if everyone just used one of the big three (Bloc, Provider, or redux) I would be happy :)

