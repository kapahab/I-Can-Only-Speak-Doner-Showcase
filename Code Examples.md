🍳 Adding Ingredients to the Plate
This method handles the core player interaction of adding a selected food item to the plate. It includes safety checks for input and stock levels, manages visual and audio feedback, and updates the global ingredient lists.

Key Features:

Validates player input and current food stock levels.

Updates the game's visuals (PutFoodOnPlate) and triggers sound effects.

Automatically deducts from the inventory stock once added.

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
🧑‍🤝‍🧑 Weighted Random Customer Selection
To ensure varied gameplay, customer types are selected using a weighted probability system. This function evaluates a pool of customers, checks if they are unlocked based on the current day, and rolls a random number against their cumulative spawn weights to choose the next customer.

    GameObject GetRandomCustomer()
    {
    int totalWeight = 0;

    foreach (var customer in randomCustomersPool)
    {
        if (gameFlow.dayCount >= customer.unlockDay)
        {
            totalWeight += customer.spawnWeight;
        }
    }

    if (totalWeight == 0) return defaultCustomerPrefab;

    int randomRoll = Random.Range(0, totalWeight);

    foreach (var customer in randomCustomersPool)
    {
        if (gameFlow.dayCount >= customer.unlockDay)
        {
            if (randomRoll < customer.spawnWeight)
            {
                return customer.prefab != null ? customer.prefab : defaultCustomerPrefab;
            }
            randomRoll -= customer.spawnWeight;
        }
    }

    return defaultCustomerPrefab; 
}
📝 Dynamic Order Generation
This utility method dynamically generates customer orders without duplicates. It takes a desired amount of ingredients and randomly selects them from a provided pool, utilizing a HashSet to guarantee that no ingredient is picked twice for the same order.

Key Features:

Automatically clamps the requested amount to prevent out-of-bounds errors.

Uses a HashSet to efficiently check for and prevent duplicate selections.

Populates both the customer's correct type index and the total order list.

    void GenericIngredientSelector(int amount, List<IngredientType> type , List<IngredientType> correctTypeIndex) 
    {
        if (type.Count == 0)
            return;

        if (amount > type.Count)
        {
            Debug.LogWarning($"Attempted to select {amount} items from a list of only {type.Count}. Clamping amount to match list size.");
            amount = type.Count;
        }

        HashSet<int> usedIndexes = new HashSet<int>();
        for (int i = 0; i < amount; i++)
        {
            int index;
            IngredientType ingredientType;
            do
            {
                index = Random.Range(0, type.Count);
                ingredientType = type[index];

            } while (usedIndexes.Contains(index) || totalOrderList.Contains(ingredientType)); // Ensure no duplicates

            usedIndexes.Add(index);
            correctOrders.Add(index);
            totalOrderList.Add(ingredientType);
            correctTypeIndex.Add(type[index]);
        }
        new List<int>(usedIndexes); 

    }
