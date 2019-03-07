#Card Controller

CardController is a class placed on every card that gets instantiated into the game world.  It allows the cards to interact with the UI and apply damage/healing/etc to any player or enemy depending on the scene.

##Public Variables

###parentToReturnTo

```csharp
public Transform parentToReturnTo;
```

This transform is used to keep track of where a card should return do if the drag and drop operation does not find a valid target.  Currently it does nothing and idk why @Alex help

###cardData

```csharp
public CardData cardData;
```

This is the part of the card that gets instantiated from its corresponding JSON file.  It contains all data on the effects of the card, as well as it's name/art/etc.

##Private Variables

###playerController

```csharp
private PlayerController playerController;
```

This is used to hold a reference to our local player controller.

###enemyController

```csharp
private EnemyController enemy;
```

This is used to hold a reference to the enemy controller.

###gameController

```csharp
private GameController gameController;
```

This is used to hold a reference to the game controller.

###classController

```csharp
private ClassController classController;
```

This is used to hold a reference to the class controller for our local player.

###customClassProcessor

```csharp
private IClassProcessor customClassProcessor;
```

This is used to hold a reference to the custom class processor for our local player.

###nextAttackDisplay

```csharp
private Text nextAttackDisplay;
```
This is used to hold a reference to the text component where we display updates on the enemy's next attack.

##Unity Default Methods

###Start();

```csharp
void Start ()
{
	//Set the game controller
	gameController = GetGameController();
	//Check if the game is running
	if(gameController.IsRunning())
	{
		//Find various controllers
		playerController = GetPlayerController();
		customClassProcessor = GetClassProcessor(playerController);
		classController = GetClassController();
		enemyController = GetEnemyController();
		//Find display for not enough energy updates
		nextAttackDisplay = GetNextAttackDisplay();
		//Set static card data
		cardData.keepInHand = false;
		cardData.removeAtEndOfCombat = false;	
	}
}
```

On start, the CardController will first check if the game is running.  If it is, it will then assign all of its references to the listed game objects ([`playerController`](#card-controller-private-variables-playercontroller), [`enemyController`](#card-controller-private-variables-enemycontroller), etc), as well as set some static card data.

##Public Methods

###OnMouseEnter();

```csharp
public void OnMouseEnter()
{
    transform.localScale += new Vector3(1.1F, 1.1f, 1.1f);
}
```

This is used to update the scale of the card when we mouse over it.

###OnMouseExit();

```csharp
public void OnMouseExit()
{
   transform.localScale = new Vector3(1, 1, 1);
}
```

This is used to update the scale of the card when we mouse out of it.

###OnBeginDrag(PointerEventData eventData);

```csharp
public void OnBeginDrag(PointerEventData eventData)
{
    parentToReturnTo = transform.parent;
    transform.SetParent(transform.parent.parent);
}
```

This method is called whenever we begin dragging a card.  It set's the [`parentToReturnTo`](#card-controller-public-variables-parenttoreturnto) so that the card can return to its spot if we don't target an enemy, and then moves the card up a level in the heirarchy to allow it to display outside of the preset hand area.

###OnDrag(PointerEventData eventData);

```csharp
public void OnDrag(PointerEventData eventData)
{
    transform.position = eventData.position;
}
```

This method is called every frame where the card is being dragged, and it assigns the cards location to that of our mouse cursor.

###OnEndDrag(PointerEventData eventData);

```csharp
public void OnEndDrag(PointerEventData eventData)
{
   //Make sure player has enough energy and is allowed to play
    if (CheckIfPlayerCanPlay())
	{
		gameController.PlaySFX(8);
		if(cardData.type == "Ongoing")
		{
			customClassProcessor.ExecuteCard(cardData, playerController, enemyController, gameController);
		}
		else
		{
			ProcessCardEffects();
		}
		customClassProcessor.ExecuteOngoing(cardData, playerController, enemyController, gameController);
		//Update player energy
		playerController.SpendEnergy(cardData.energyCost);
		if(!cardData.keepInHand)
		{
			//Discard the card and remove it from the game world
			if(!cardData.keepFromDiscard)
			{
				playerController.discard.Add(cardData);
			}
			playerController.hand.Remove(cardData);
			Destroy(gameObject);
		}
		transform.SetParent(parentToReturnTo);
        //GetComponent<CanvasGroup>().blocksRaycasts = true;
    }
	//If player can play but doesn't have enough energy to play the card they clicked, let them know
	else if(playerController.canPlay)
	{
		UpdateInfo("Not enough energy!");
        transform.SetParent(parentToReturnTo);
		gameController.PlaySFX(9);
        // GetComponent<CanvasGroup>().blocksRaycasts = true;
    }
}
```

This method is called whenever we stop dragging a card.  It does a lot of stuff.  It first checks to ensure the player can play with [`CheckIfPlayerCanPlay()`](#card-controller-private-methods-checkifplayercanplay).  It then checks on the type of card played - if it's an ongoing card, it instantly sends it to the [`customClassProcessor`](#card-controller-private-variables-customclassprocessor) .  Otherwise, it process the card effects via [`ProcessCardEffects()`](#card-controller-private-methods-processcardeffects).  It will then use the [`customClassProcessor`](#card-controller-private-variables-customclassprocessor) to execute any static ongoing effects, use the [`playerController`](#card-controller-private-variables-playercontroller) to reduce the players energy, and then begin to discard the card.  If the player doesn't have enough energy to play the card, the final `else if` will trigger, returning the card to the players hand and alerting them to their lack of energy.

###AddToHand();

```csharp
public void AddToHand()
{
	gameController.AddCardToHand(cardData);
}
```

This method simply uses the [`gameController`](#card-controller-private-variables-gamecontroller) to add the current card to the players hand.

###AddToHandNoHacking();

```csharp
public void AddToHandNoHacking()
{
	gameController.AddCardToHand(cardData);
	gameController.CloseInbetween();
}
```

This method simply uses the [`gameController`](#card-controller-private-variables-gamecontroller) to add the current card to the players hand, but then also requets that the card adding interface be closed right afterwards.  This is used to add cards using the hackermode object without needing to adjust much else and is only temporary.

###DrawFromDiscard();

```csharp
public void DrawFromDiscard()
{
	gameController.MoveCard(cardData, playerController.discard, playerController.hand);
	gameController.KillHacker();
}
```

This method is used to move a card from the plaers discard pile to their hand.  It uses the [`gameController`](#card-controller-private-variables-gamecontroller) to accomplish this, as well as closing the hacker mode menu for now.

###UpdateInfo(string x);

```csharp
public void UpdateInfo(string x)
{
	nextAttackDisplay.text = x;
	//Clear text in two seconds
	StartCoroutine(ClearInfo(2, x, nextAttackDisplay));
}
```

This method is used to update the [`nextAttackDisplay`](#card-controller-private-variables-nextattackdisplay) and have it display text about the incoming attack, as well as launch a coroutine which will fire after 2 seconds and return the text display to it's original state.

##Private Methods

###CheckIfPlayerCanPlay();

```csharp
private bool CheckIfPlayerCanPlay()
{
	if (playerController.playerData.energy >= cardData.energyCost && playerController.canPlay)
	{
		return true;
	}
	else
	{
		return false;
	}
}
```

This method is used to check if the player is allowed to play.  It makes sure they have enough energy to spend on the current card, and also checks that they aren't disabled form playing for some other reason (spectating, inbetween rounds, etc).

###ProcessCardEffects();

```csharp
private void ProcessCardEffects()
{
	//Increment attacksThisTurn if its an attack
	if(cardData.type == "Attack")
	{
		playerController.attacksThisTurn += 1;
	}
	//Apply card damage
	if(cardData.damage != 0)
	{
		//Check for stealth and update damage accordingly
		if(customClassProcessor.BoolHelpers("GetStealth"))
		{
			int y = customClassProcessor.IntHelpers("GetStealthBonus");
			y = cardData.damage + y;
			enemyController.TakeDamage(y);
		}
		else
		{
			enemyController.TakeDamage(cardData.damage);
		}
	}
	//Apply AOE damage
	if(cardData.aoeDamage != 0)
	{
		//EnemyController[] enemies = FindObjectsOfType<EnemyController> ();
		foreach(EnemyController e in FindObjectsOfType<EnemyController> ())
		{
			e.TakeDamage(cardData.aoeDamage);
		}
	}
	//Apply card shields
	if(cardData.gainShields != 0)
	{
		playerController.GainShields(cardData.gainShields);
		gameController.PlaySFX(5);
	}
	//Draw cards
	if(cardData.drawCards != 0)
	{
		gameController.DrawCards(cardData.drawCards);
		gameController.PlaySFX(7);
	}
	//Gain Energy
	if(cardData.youGainEnergy != 0)
	{
		playerController.GainEnergy(cardData.youGainEnergy);
	}
	//Allows Move
	if(cardData.allowsMove)
	{
		gameController.AllowMove();
		gameController.PlaySFX(6);
	}
	//Poison a bitch
	if(cardData.applyPoison)
	{
		//Poison a bitch
	}
	//Gain energy if poisoned
	if(cardData.ifPoisonedGainEnergy != 0)
	{
		playerController.GainEnergy(cardData.ifPoisonedGainEnergy);
	}
	//Draw from top of discard
	if(cardData.drawFromTopOfDiscardPile != 0)
	{
		//put card from top of discard into hand
	}
	//Get some gold
	if(cardData.goldToGain != 0)
	{
		playerController.GainGold(cardData.goldToGain);
	}
	//Run special logic
	if(cardData.isSpecial)
	{
		customClassProcessor.ExecuteCard(cardData, playerController, enemyController, gameController);
	}
	//Remove From Combat
	if(cardData.removeFromCombat)
	{
		//Add logic to stop card from being drawn this combat.  Will require updates zo the
		//actual draw method in order to check for a new "isDrawable" property
	}	
}
```

This method is responsbile for executing all card effects.  These effects are all stored in [`cardData`](#card-controller-public-variables-carddata).  First, we check to see what type of card we are playing.  Currently we use this information to keep track of how many attack cards a player has played in a turn.  It then goes doesn the line of shared card effects 1 by 1 to apply the effects.  It will finish off by sending the card to the [`customClassProcessor`](#card-controller-private-variables-customclassprocessor) if [`cardData`](#card-controller-public-variables-carddata) indicates that this is required.  Lastly, it will remove the card from combat, if the card is flagged as such.

###GetGameController();

```csharp
private GameController GetGameController()
{
	GameController newController = GameObject.Find("controller").GetComponent<GameController> ();
	return newController;
}
```

This method is used to find and return the [`gameController`](#card-controller-private-variables-gamecontroller).

###GetPlayerController();

```csharp
private PlayerController GetPlayerController()
{
	PlayerController newPlayerController = GameObject.Find(PhontonNetwork.NickName).GetComponent<PlayerController> ();
	return newPlayerController;
}
```

This method is used to find and return the [`playerController`](#card-controller-private-variables-playercontroller).

###GetClassController();

```csharp
private ClassController GetClassController()
{
	ClassController newClassController = GameObject.Find(PhotonNetwork.NickName).GetComponent<ClassController> ();
	return newClassController;
}
```

This method is used to find and return the [`classController`](#card-controller-private-variables-classcontroller).

###GetEnemyController();

```csharp
private EnemyController GetEnemyController()
{
	EnemyController newEnemyController = GameObject.Find("Enemy").GetComponent<EnemyController> ();
	return newEnemyController;
}
```

This method is used to find and return the [`enemyController`](#card-controller-private-variables-enemycontroller).

###GetNextAttackDisplay();

```csharp
private Text GetNextAttackDisplay()
{
	Text theDisplay = GameObject.Find("nextAttackDisplay").GetComponent<Text> ();
	return theDisplay;
}
```

This method is used to find and return the [`nextAttackDisplay`](#card-controller-private-variables-nextattackdisplay).

###GetClassProcessor(PlayerController thePlayer);

```csharp
private IClassProcessor GetClassProcessor(PlayerController thePlayer)
{
	IClassProcessor newProcessor = thePlayer.gameObject.GetComponent<IClassProcessor> ();
	return newProcessor;
}
```

This method is used to find and return the [`customClassProcessor`](#card-controller-private-variables-customclassprocessor).

##Coroutines

###ClearInfo(int amt, string x, Text t);

```csharp
IEnumerator ClearInfo(int amt, string x, Text t)
{	
	//Wait however long
    yield return new WaitForSeconds(amt);
	//If the text hasn't been updated while waiting, set it back to the attack information
	if(x == t.text)
	{
		t.text = gameController.attackInfoStr;
	}
}
```

This coroutine is used to clear temporary information from the [`nextAttackDisplay`](#card-controller-private-variables-nextattackdisplay) and set it back to the reported information from the [`gameController`](#card-controller-private-variables-gamecontroller).