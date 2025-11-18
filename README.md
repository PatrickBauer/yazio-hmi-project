# Tasks

## Step 1

- Load recipe data to be displayed (inside fetchRecipes())
- URL: https://dummyjson.com/recipes?limit=18

## Step 2

- Add error handling (after the URL changed) to fetchRecipes()

## Step 3

- Improve the recipe list display (e.g. two columns and adding a gap)

## Step 4

- The title of recipes does not seem to work
- Find out why and fix the title display

## Step 5

- What if the external JSON endpoint changes its schema in the future?
- How could we ensure to log and show an error message instead of having broken items being shown?

## Step 5

- Clicking on the heart icon to add a favorite does not work as expected
- When reloading, any heart that has been clicked should still be filled, but its not
- Can you find and fix the problem?

## Step 6

- Check the ingredients list of each recipe
- There are two prepared functions: formatIngredients() and concatenateIngredients()
- Add appropriate types to those
- Implement the inner logic of concatenateIngredients()

- Add error handling to concatenateIngredients() and let "formatIngredients" gracefully handle that error
- change the concatenateIngredients input to type "any" and pass the number 5 from formatIngredients()
- Lets talk about this - what options do we have for this? Which do you prefer?
