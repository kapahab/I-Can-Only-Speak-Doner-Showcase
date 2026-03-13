Below are some code and system examples used in the project:

void AddFood()
{
    if (!thisFoodSelected)
        return;
    if (isIngredientAdded)
        return;
    if (!InputManager.Instance.InteractPressed())
        return;

    if (foodAvailability != null)
        foodAvailability.CheckFoodAvailability();
    else
    {
        Debug.LogWarning("FoodAvailibilityManager component is missing. Assuming food is available.");
        foodAvailable = true; // Default to true if the component is missing
    }

    if (!foodAvailable)
        return;

    foodOnPlate.PutFoodOnPlate(cloneObj, gameFlow.xPosOfPlate, gameFlow.yPosOfPlate); //add food visuals
    gameFlow.AddIngredientToPlayerList(ingredientName);
    isIngredientAdded = true;
    OnFoodAdded?.Invoke();

    if (foodAvailability != null) //this logic moved into foodAvailability's CheckFoodAvailability method
        foodAvailability.DecreaseFoodStock();
    else
        Debug.LogWarning("FoodAvailibilityManager component is missing. Cannot decrease food stock.");

    AudioManager.Instance.PlaySound(soundType);

    foodAvailability.CheckFoodAvailability(); //in case it ran out of stock after decreasing
}
