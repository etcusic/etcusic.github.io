---
layout: post
title:      "DRY Dynmaic Form in React"
date:       2021-03-17 16:24:02 +0000
permalink:  dry_dynmaic_form_in_react
---


Context:
This is an extenuation of my recipes/ingredients project. The challenges that this post addresses is having a React form that handles nested attributes (ingredients), can dynamically add and remove nested attributes, and that can handle both creating *and* updating new recipes - basically making our frontend operation as DRY as possible. 

So, let's begin with what what props need to be sent with the `<RecipeForm />` component. Since this form needs to be able to handle creating a recipe or updating an existing recipe, this component needs to receive two parameters - a recipe and an action. The recipe will either be an existing one for update, or an object with all the necessary keys but empty values for a new one. The action will then be either a 'POST' or 'PATCH' request sent to the backend. Here is the setup function for implementing the form component (I use state from its parent component to set it as the view):
```
recipeForm = (recipe, action) => {
  this.setState({
    view: <RecipeForm 
      userId={ this.state.userId } 
      recipe={ recipe } 
      ingredients={ this.state.pantry } 
      sendRecipeData={ action }
    />
  })
}
```
As you can see the recipe and action are sent through as parameters and then as props to the form component. Also, the ingredients from the pantry are sent through so that we can select pre-existing ingredients for our recipe. This way we can call the form form various places within the application, and it can handle both creating a new recipe or update an existing one. Once the component gets mounted, I apply all the recipe data to the form component's state:
```
componentDidMount(){
    this.setState({
        name: this.props.name,
        servings: this.props.servings,
        instructions: this.props.instructions,
        ingredients: this.props.ingredients
    })
}
```
Setting these attributes as state will then allow me to handle any changes within the form using React's `onChange` event handler. These changes will update the state, and then we can package state when the form needs to be submitted with the final set of data. So now we just need to set up some functions to process any changes made within the form. We'll begin by setting up the function to handle all non-ingredient input. Thanks to JavaScript's block notation we can set up just one function to handle this:
```
changeRecipe = (event, category) => {
    let recipeObject = this.state.recipe
    recipeObject[category] = event.target.value
    this.setState({ recipe: recipeObject })
}
```
We begin by making a copy of state. This way we don't alter state directly as we handle the change. Of course the event is the first parameter, and then we have the category as the second. The category represents the key to the recipe/state object, which can then be accessed using block notation to apply the value of the target. Here is what the recipe's name input element looks like:
```
<label>Recipe Name:</label>
<input type ="text" value={ this.state.name } onChange={event => this.changeRecipe(event, "name")}>
```
Now for the nested attributes - ingredients. This part is slightly more involved because it involves dynamically incorporating a component that can be added or subtracted from the form multiple times over. To begin, let's take a look at the state.ingredients array. This as an arry of ingredient objects with an id, name, and quantity. These ingredients are then rendered by being mapped and passed down into a `<Ingredient />` component. I decided to write out a function to handle making each ingredient component so that they can be appropriately indexed and accessed easily. Here is the function and how the ingredients are rendered:
```
makeComponent = (ing, i) => {
    return <Ingredient 
        keyId={i + 1} 
        ingredientName={ ing.name } 
        ingredientQuantity={ ing.quantity } 
        ingredients={ this.props.ingredients } 
        changeIngredient={ this.changeIngredient }
    />
}

// inside of render() 
<div id="new-recipe-ingredients">
    { this.state.ingredients.map((ing, i) => this.makeComponent(ing, i)) }
</div>
```
The `keyId` is handy for accessing that ingredient in the `changeIngredient` function, which is very similar to `changeRecipe` except that it also takes an id parameter to speedily access the required ingredient in the `state.ingredients` array.
```
changeIngredient = (event, id, category) => {
    let recipeObject = Object.assign({}, this.state.recipe)
    recipeObject.ingredients[id - 1][category] = event.target.value
    this.setState({ recipe: recipeObject })
}
```
The area I had the most trouble in throughout this process was having preselected ingredients to account for a pre-existing recipe with ingredients that can be changed. It saddens me to say that I needed to use an if statement in the render function. Sad face. I'm sure there is a way to tighten this up, but I haven't gotten to that just yet, so here it is in its current state:
```
<select name="ingredients" id="new-recipe-ingredients" onChange={event => this.props.changeIngredient(event, this.props.keyId, "id") }>
    { this.props.ingredients.map(ingredient => {
        if (ingredient.name === this.props.ingredientName){
            return <option selected key={`ingredient-option-${ingredient.id}`} value={ ingredient.id } >{ ingredient.name }</option>
        } else {
            return <option key={`ingredient-option-${ingredient.id}`} value={ ingredient.id }>{ ingredient.name }</option>
        }
    })}
</select>
```
And that's most of the form component. There are also add and delete ingredient component functions, but those are very straight forward, and in the interest of time, I figured we'd jump to our action, then we're done! For the action, it needs to take a few parameters. Five actually. First, of course, is the event. Then we need a method ('POST' or 'PATCH'), a recipe object to send, the userId, and lastly the reaction, which will be a function to tell our application what to do with our return info. And so once we send our form it hits this aciton function:
```
export const sendRecipeData = (event, method, recipe, userId, reaction) => {
    event.preventDefault()
    let id = method === "PATCH" ? `/${recipe.id}` : ""
    const route = `http://localhost:3001/users/${userId}/recipes` + id
    const configObject = {
        method: method,
        headers: {
            "Content-Type": 'application/json',
            "Accept": 'application/json'
        },
        body: JSON.stringify(recipe)
    }
    fetch(route, configObject)
      .then(response => response.json())
      .then(json => reaction(json.id))
}
```
And there you have it. Our dynamic, super DRY frontend form using React!
