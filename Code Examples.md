Below are some examples of code from the project:
💾 Data Management: Ingredient Scriptable Objects
To keep the game easily tunable and data-driven, ingredient properties are stored using Unity's ScriptableObject system. The FoodData class centralizes everything from stock levels and economy costs to minigame mechanics (like the memory reveal duration), making it easy to create and balance new ingredients without touching code.

Key Features:

Centralized economy data (unlock costs, restock costs, sell costs).

Built-in methods to calculate dynamic costs based on missing stock.

Tracks specific minigame states, such as how long an item remains revealed.

    [CreateAssetMenu(fileName = "FoodData", menuName = "Scriptable Objects/FoodData")]
    public class FoodData : ScriptableObject
    {
    public IngredientType ingredientType;
    public bool availableInDemo = true;
    public int buyableOnDay = 1;
    public bool unlocked;
    public int stock;
    public int maxStock = 10;
    public int unlockCost = 100;
    public float stockCost = 1; // Cost per unit to restock


    public float CurrentStockCost()
    {
        return (maxStock - stock) * stockCost;
    }
    public float sellCost;

    public Sprite sprite;

    public string gibberishName;

    public LocalizedString localizedString;

    public int revealDuration = 2; //how many days it will be revealed for each time its guessed correctly in the memory game

    public int revealedFor; // How many days this has been revealed for, for accounting

    public void RevealedCorrectly()
    {
        revealedFor =+ revealDuration;
    }

    public void EndDayRevealAccount()
    {
        if (revealedFor > 0)
        {
            revealedFor--;
        }
    }
    }
📡 Event-Driven Game Logic
The EventManager acts as a central hub for broadcasting player actions to the rest of the game. By using C# delegates and events, the input logic is decoupled from the game systems that need to respond to those inputs (like animations, UI updates, or scoring).

Key Features:

Handles complex input interactions, like the "hold-to-serve" mechanic using timers.

Broadcasts specific events (e.g., OnFoodTrashed, OnPlateServed) for other scripts to listen to.

Safely manages screen transitions using Coroutines to prevent frame-perfect glitches.

    public class EventManager : MonoBehaviour
    {

    public delegate void ResetFoodMaking();
    public static event ResetFoodMaking OnResetFoodMaking;

    public delegate void DonerEnter();
    public static event DonerEnter OnDonerEnter;
    private bool inDonerMinigame = false;

    public delegate void DonerExit();
    public static event DonerExit OnDonerExit;

    public delegate void ServePlate();
    public static event ServePlate OnPlateServed;


    public delegate void ScreenSwitchToCustomer();
    public static event ScreenSwitchToCustomer OnScreenSwitchToCustomer;

    public delegate void FoodTrashed();
    public static event FoodTrashed OnFoodTrashed;

   

    FoodOnPlate foodOnPlate;
    [SerializeField] TutorialManager tutorialManager;

    float startTime = 0f;
    public float holdTime = 1.5f;


    [SerializeField] Animator handAnimator;




    // Start is called once before the first execution of Update after the MonoBehaviour is created
    void Start()
    {
        foodOnPlate = GetComponent<FoodOnPlate>();
        startTime = float.PositiveInfinity;
    }

    // Update is called once per frame
    void Update()
    {
        if (gameFlow.currentGameState != GameState.KitchenScreen)
            return;

        if (InputManager.Instance.TrashPressed())
        {
            if (gameFlow.carbList.Count != 0 || gameFlow.toppingList.Count != 0 || gameFlow.sauceList.Count != 0 || gameFlow.spiceList.Count != 0 || gameFlow.donerList.Count != 0)
            {
                OnFoodTrashed();
                
                OnResetFoodMaking();

            }
        }

       

        if (InputManager.Instance.UpButtonPressed())
        {
            if (ZonePicker.currentActiveZone == null)
                StartCoroutine(WaitAndSwitchScreen());
        }



        if (ZonePicker.currentActiveZone != null)
        {
            return;
        }


        if (gameFlow.carbList.Count == 0 && gameFlow.toppingList.Count == 0 && gameFlow.sauceList.Count == 0 && gameFlow.spiceList.Count == 0 && gameFlow.donerList.Count == 0)
            return;


        if (InputManager.Instance.InteractPressed())
        {
            startTime = Time.time;
        }

        if (InputManager.Instance.interactAction.IsPressed())
        {
            if (Time.time - startTime >= holdTime)
            {

                StartCoroutine(WaitAndSwitch());

                OnResetFoodMaking();
                startTime = float.PositiveInfinity; // to prevent multiple triggers
            }
        }

        else
        {
            startTime = float.PositiveInfinity;
        }
    }


        IEnumerator WaitAndSwitch()
    {
        yield return new WaitForSeconds(0.01f);
        OnPlateServed();
        OnScreenSwitchToCustomer();
    }

    IEnumerator WaitAndSwitchScreen()
    {
        yield return new WaitForSeconds(0.01f);
        OnScreenSwitchToCustomer();
        Debug.Log("Screen switched to customer");
    }
    }
🕹️ Game State Management
To cleanly track what phase of gameplay the player is currently experiencing, the project relies on a GameState enum. This allows various managers to quickly check the current state and execute logic accordingly (for example, preventing kitchen inputs while the game is paused or on the drinks screen).

    public enum GameState
    {
        GamePause,
        KitchenScreen,
        CustomerScreen,
        DrinksScreen,
        CustomerTrashSelection,
        DayTransition,
        PartyNight,
        LoseScreen
    }
    
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


Most of the code could be better arranged, as I was not that experienced while making this project, but it really thought me what to do wrong and what to do right!
