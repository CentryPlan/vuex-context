# vuex-context

Write fully type inferred Vuex modules:
- Extremely lightweight
- No boilerplate
- No class
- Handles module refactoring

## Install

1. Install vuex
`npm install vuex --save`

2. Install module:
`npm install vuex-context --save`

## Usage

1. Write your Vuex modules in the standard Vuex way:

```ts
export interface CounterState {
  count: number;
}

export const namespaced = true;

export const state = (): CounterState => ({
  count: 0
});

export const mutations = {
  increment(state: CounterState) {
    state.count++;
  },
  incrementBy(state: CounterState, payload: number) {
    state.count += payload;
  }
};

export const actions = {
  async incrementAsync(context) {
    // ...
  }
};

export const getters = {
  doubleCount(state: CounterState): number {
    return state.count * 2;
  },
  quadrupleCount(state: CounterState, context): number {
    // ...
  }
}

```

2. Create the module's context

```ts
export const Counter = Context.fromModule({
  state,
  mutations,
  actions,
  getters
});
```

3. That's it, now you have access to a completely typed version of your module

```ts
// Use store and the namespace leading to the module to get a new context instance
const counterModule = Counter.getInstance(store, 'counter');
counterModule.dispatch.incrementAsync();

// Or use the context to access the current scope
export const actions = {
  async incrementAsync(context) {
    const counterModule = Counter.getInstance(context);
    counterModule.commit.increment();
  }
};

// You can also type your getters
export const getters = {
  doubleCount(state: CounterState): number {
    return state.count * 2;
  },
  quadrupleCount(state: CounterState, context): number {
    const getters = Counter.getGetters(context);
    return getters.doubleCount * 2;
  }
}

```


## Circular references

**Warning**: Be careful when returning values from your **actions** and **getters**!

```ts
export const actions = {
  async incrementAsync(context) {
    const counterModule = Counter.getInstance(context);
    counterModule.commit.increment();
    // Circular reference here, as incrementAsync needs the type from counterModule and counterModule needs the type from incrementAsync
    // Result: counterModule is cast to any
    return counterModule.state.count;
  }
};
```

To avoid this, you should manually write the return types of your actions and getters:

```ts
export const actions = {
  // Specify the return type here
  async incrementAsync(context): Promise<number> {
    const counterModule = Counter.getInstance(context);
    counterModule.commit.increment();
    // Everything works fine now
    return counterModule.state.count;
  }
};
```

## Contributors

If you are interested and want to help out, don't hesitate to contact me or to create a pull request with your fixes / features.

The project now also contains samples that you can use to directly test out your features during development.

1. Clone the repository

2. Install dependencies
`npm install`

3. Launch samples
`npm run serve`

4. Launch unit tests situated in *./tests*. The unit tests are written in Jest.
`npm run test:unit`


## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details
