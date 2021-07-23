# reactShoppingCart
This React shopping cart fetches and displays items. Item counts can be increased and is reflected in the navbar. Total is auto calculated based on cart items. Cart can be cleared as well.

A walkthrough of the React code is given below.

To start with the useReducer is setup inside of context.js.
```React
  const [state, dispatch] = useReducer()
```

Next pass in initial state into useReducer.
```React
const initialState = {
  loading:false,
  cart: cartItems,
  total: 0,
  amount: 0
}

  const [state, dispatch] = useReducer(initialState);
```  

Next set up the reducer boilerplate in reduce.js.
```React
const reducer = (state, action) => {
  return state;
};

export default reducer;
```

Then import the reducer into context.js (not shown). Next use reducer as the first argument inside of useReducer. 
```React
  const [state, dispatch] = useReducer(reducer, initialState);
```

An error will occur because 'cart' is undefined since it was the previous state value. Now the state value is state. So change the value using the spread operator from cart to state which makes it available throughout the app where globalContext is used(state includes: 'loading', 'cart', 'total', and 'amount').
```React
    <AppContext.Provider
      value={{
        ...state,
      }}
    >
```

Next the navbar is connected to the cart to monitor the items count of the cart.
```React
  const {amount} = useGlobalContext()
  
   <p className="total-amount">{amount}</p>
```

Next the loading state variable is connected to the App.js file to render a loading screen during an API fetch or whenever loading is set to true inside of context.js.
```React
  const {loading} = useGlobalContext()
  if (loading) {
    return (
      <div className='loading'>
        <h1>Loading...</h1>
      </div>
    )
  }
```

Next the value of the total is retrieved inside the CartContainer.
```React
  const { cart, total } = useGlobalContext()
  
            <h4>
            total <span>${total}</span>
          </h4>
```

Now the initial functionality of useReducer has been set up. Next functionality is addressed. In context.js a function to dispatch an action is setup. Then that function will be passed to the provider which will provide it to whatever component needs that functionality. To start with the clear cart button functionality is set-up in context.js and passed down into the Provider to be available throughout the app.
```React
  const clearCart = () => {
    dispatch({ type: "CLEAR_CART" });
  };

  return (
    <AppContext.Provider
      value={{
        ...state,
        clearCart,
      }}
    >
```

clearCart is next brought into CartContainer via destructuring and used inside of the clear cart button.
```React
  const { cart, total, clearCart } = useGlobalContext();
  
          <button className="btn clear-btn" onClick={clearCart}>
          clear cart
        </button>
```

For the button to be made to work, it needs to be setup inside the reducer.js file where only the cart state value is changed to an empty array.
```React
  if (action.type === "CLEAR_CART") {
    return { ...state, cart: []};
  }
```

At this point the clear cart button should be working as intended. Spreading out the state values above prvents them from being removed from the state when useReducer returns the action - only the cart array was changed to an empty array.

Next the remove single item functionality is made by creating another new function inside of context.js and passed down to the provider just as for clear cart.
```React
  const remove = (id) => {
    dispatch({ type: "REMOVE", payload: id });
  };

  return (
    <AppContext.Provider
      value={{
        ...state,
        clearCart,
        remove
      }}
    >
```

The remove function will be needed inside of cardItem.js
```React
  const {remove} = useGlobalContext();
  
          <button className="remove-btn" onClick={() => remove(id)}>
          remove
        </button>
```

To make the button work, it must once again be set up inside of reducer.js. state.cart filters out the card items from the old state whose id does not match the id attached to the payload.
```React
  if(action.type === 'REMOVE'){
      return {...state, cart: state.cart.filter((cartItem)=> cartItem.id !== action.payload)}
  }
```

At this point the remove item button should be working as intended. 
