# I Can Only Speak Doner 🥙

https://youtu.be/i9vNcCyRj18?si=Nyt1hE1ieaFeG00H

**I Can Only Speak Doner** is a 2D pixel art cooking and language-learning game developed in Unity (C#). Inspired by titles like *Cook, Serve, Delicious!* and *Papers, Please*, players take on the role of an expat working in a foreign country where they do not speak the local language. 

This repository highlights the core architectural decisions, custom tooling, and gameplay programming that brought the project to life. The whole project cannot be shared as it's a commercial product. This repository is for portfolio purpouses. I was the only programmer in the project.

## 🎮 Core Mechanics & Implementation

* **Language Guessing & Order System:** The core gameplay loop revolves around a non-verbal communication system. Players hand customers a menu, observe what they point at, and take notes. I implemented a strict validation system where incorrect notes trigger immediate visual feedback (finger shaking) from the customer AI.
* **Cooking Pipeline:** [Briefly describe how you structured the code for assembling the doner/cooking process].




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


## 🛠️ Tech Stack & Architecture

* **Engine:** Unity
* **Language:** C#
* **Art:** Aseprite (2D Pixel Art)
* **Custom Tooling:** Python (for automated localization pipelines)

### Highlighted Coding Architectures
* **[Insert Pattern Name, e.g., State Machine]:** Used to manage the customer flow (entering, waiting, ordering, reacting, leaving).
* **[Insert Pattern Name, e.g., Scriptable Objects]:** Utilized for managing the menu items, language variations, and customer data efficiently without hardcoding.

## ⚙️ Custom Tooling: Automated Localization
To handle multiple languages (Turkish, German, Spanish, Portuguese), I built a custom Python script that leverages API integration to automatically parse, translate, and reformat CSV and JSON files directly into the game's architecture. 

## 📂 Repository Structure
* `/Assets/Scripts/Core` - Main game loop and managers.
* `/Assets/Scripts/Entities` - Customer AI and interaction logic.
* `/Tools/Localization` - Python automation scripts.

## 🚀 Play the Game
A playable demo is currently available! 
[Link to Steam Page]
