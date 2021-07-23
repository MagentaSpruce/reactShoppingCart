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

At this point the remove item button should be working as intended. Next the button incrementer will be worked on starting with the increase. Again the process starts with a new function added into the AppProvider.
```React
  const increase = (id) => {
    dispatch({type: 'INCREASE', payload: id})
  }
  const decrease = (id) => {
    dispatch({type: 'DECREASE', payload: id})
  }
  
      <AppContext.Provider
      value={{
        ...state,
        clearCart,
        remove,
        increase,
        decrease
      }}
    >
```

Next both are destructured in the cardItem and added to the relevant buttons. Using spread operator on cartItem copies all the properties and so only the amount property is changed by +1.
```React
  const { remove, increase, decrease } = useGlobalContext();
      
      <button className="amount-btn" onClick={() => increase(id)}>
      <button className="amount-btn" onClick={() => decrease(id)}>
```     

Next useReduce is setup for increase.
```React
  if (action.type === "INCREASE") {
    let tempCart = state.cart.map((cartItem) => {
      if (cartItem.id === action.payload) {
        return { ...cartItem, amount: cartItem.amount + 1 };
      }
      return cartItem;
    });
    return { ...state, cart: tempCart };
  }
```  

At this point the increase button works but changes are not displayed to the navbar or total. Before those are taken care of the decrease functionality is finished installing. To prevent negative items from being possible and to remove the item on 0 the filter method is chained to return an empty cart.
```React
  if (action.type === "DECREASE") {
    let tempCart = state.cart.map((cartItem) => {
      if (cartItem.id === action.payload) {
        return { ...cartItem, amount: cartItem.amount - 1 };
      }
      return cartItem;
    }).filter((cartItem) => cartItem.amount !== 0);
    return { ...state, cart: tempCart };
  }
```

Next the values should be updated in the shooping bag and the total whenever the incrementer buttons are used. A useEffect is used to monitor changes in the cart.
```React
  useEffect(()=>{
    console.log('hello');
    
  },[state.cart])
  ```
  
  Now an action is dispatched from the useEffect.
  ```React
    useEffect(() => {
    dispatch({ type: "GET_TOTALS" });
  }, [state.cart]);
  ```
  
  Next the reducer is set up to update the amount and total.
  ```React
  if (action.type === "GET_TOTALS") {
    const { total, amount } = state.cart.reduce(
      (cartTotal, cartItem) => {
        const { price, amount } = cartItem;
        const itemTotal = price * amount;
        cartTotal.total += itemTotal;
        cartTotal.amount += amount;
        return cartTotal;
      },
      {
        total: 0,
        amount: 0,
      }
    );
    return { ...state, total, amount };
  }
```

To prevent wonky totals from cropping up the total is parsed and assigned a fix number of digits to be returned.
```React
    let { total, amount } = state.cart.reduce(.............
    
        total = parseFloat(total.toFixed(2));
```

At this point the total and amount functionality should be working. Next the ability to fetch data from an API using useReducer is set up. Two dispatches are used, one for rendering the loading screen during fetch and the other to display the fetched items sent as response.
```React
  const fetchData = async () => {
    dispatch({ type: "LOADING" });
    const response = await fetch(url);
    const cart = await response.json();
    dispatch({type: 'DISPLAY_ITEMS', payload:cart});
  };
```

To call this function another useEffect is added.
```React
  useEffect(() => {
    fetchData();
  }, []);
```

Next the reducer is set up for loading and displaying the items.
```React
  if (action.type === "LOADING") {
    return { ...state, loading: true };
  }
  
  if (action.type === "DISPLAY_ITEMS") {
    return { ...state, cart: action.payload, loading: false };
  }
```

This returns the fetched data and the rest of the functionality all works. The application is complete - however, a refactoring did occur in keeping with DRY principles.
```React
  const toggleAmount = (id, type) => {
    dispatch({ type: "TOGGLE_AMOUNT", payload: { id, type } });
  };
  
  
    const { remove, increase, decrease, toggleAmount } = useGlobalContext();
    
            <button className="amount-btn" onClick={() => toggleAmount(id, 'inc')}>
            <button className="amount-btn" onClick={() => toggleAmount(id, "dec")}>
            
      if (action.type === "TOGGLE_AMOUNT") {
    let tempCart = state.cart.map((cartItem) => {
      if (cartItem.id === action.payload.id) {
        if (action.payload.type === "inc") {
          return { ...cartItem, amount: cartItem.amount + 1 };
        }
        if (action.payload.type === "dec") {
          return { ...cartItem, amount: cartItem.amount - 1 };
        }
      }
      return cartItem;
    });
    return { ...state, cart: tempCart };
  }
```


***End walkthrough
  
