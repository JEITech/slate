#Enemy Controller

EnemyController is a class placed on every enemy that controls how they interact with the game world, including dealing/taking damage, updating certain UI elements, and triggering certain animations.

##Public Variables

###enemyData

```csharp
public EnemyData enemyData;
```

This gets instantiated from a JSON file and includes the enemy name, health, and more.

###canvas

```csharp
public GameObject canvas;
```

This holds a reference to the main canvas in which we are rendering the game.

###death

```csharp
public GameObject death;
```

This holds a reference to the canvas in which we render the death screen.

###attackData

```csharp
public EnemyAttackData attackData;
```

This holds data on the next enemy attack, and gets updated whenever a new attack is selected in [`NextAttackClientRpc()`](#enemy-controller-pun-rpcs-nextattackclientrpc).

###controller

```csharp
public GameController controller;
```

This holds a reference to the ['GameController'](#game-controller).

###startingHealth

```csharp
public int startingHealth;
```

This int is set to the enemy data health on ['Start()'](#enemy-controller-unity-default-methods-start), and is used to help animate the enemy health bar in ['AnimateHealthBar()'](#enemy-controller-public-methods-animatehealthbar).

###rounds

```csharp
public int rounds;
```

This is used to keep track of how many turns have happened over the course of the battle.

##Private Variables

###dataController

```csharp
private DataController dataController;
```

This holds a reference to the data controller.

###enemyHeaqlth

```csharp
private Text enemyHealth;
```

This is used to reference the text for displaying the enemy's current health.

###enemyDamageDisplay

```csharp
private Text enemyDamageDisplay;
```

This is used to reference the text for displaying damage that the enemy is currently taking.

###enemyNameText

```csharp
private Text enemyNameText;
```

This is used to reference the text for displaying the enemy's name.

###player

```csharp
private PlayerController player;
```

This is used to hold a reference to our local player controller.

###gameManager

```csharp
private TGGameManager gameManager;
```

This is used to reference the game manager.

###healthBar

```csharp
private Animator healthBar;
```

This is used to reference the animation controller of the enemy's health bar.

##Unity Default Methods

###Start();

```csharp
void Start ()
{
	dataController = FindObjectOfType<DataController> ();
	gameManager = FindObjectOfType<TGGameManager> ();
	gameManager.EnemyInit(dataController);
	SetEnemyStartingHealth();
	startingHealth = enemyData.health;
	rounds = 0;
	EnterLevel();
}
```

On start, we first assign our references to ['dataController'](#enemy-controller-private-variables-dataController) and ['gameManager'](#enemy-controller-private-variables-gameManager), and then we use that manager to initialize our enemy with the proper data.  From there we call ['SetEnemyStartingHealth()'](#enemy-controller-private-methods-setenemystartinghealth), set our rounds counter to 0, and call ['EnterLevel()'](#enemy-controller-public-methods-enterlevel).

##PUN Methods

###OnPhotonSerializeView();

```csharp
public void OnPhotonSerializeView(PhotonStream stream, PhotonMessageInfo info)
{
	if(stream.IsWriting)
	{
		stream.SendNext(enemyData.enemyName);
		stream.SendNext(enemyData.health);
		stream.SendNext(enemyData.shields);
	}
	else
	{
		this.enemyData.enemyName = (string)stream.ReceiveNext();
		this.enemyData.health = (int)stream.ReceiveNext();
		this.enemyData.shields = (int)stream.ReceiveNext();
		UpdateShields(this.enemyData.shields);
		UpdateName(this.enemyData.enemyName);
	}
}
```

This is the standard pun method that gets called whenever one of the indicated values is changed, and sends that update to all clients.

##Public Methods

###UpgrageHealth();

```csharp
public void UpgradeHealth()
{
    rounds = rounds + 1;
	int bonus = rounds * 40;
	PlayerController[] allPlayers = FindObjectsOfType<PlayerController>();
	int bonus2 = 60 * allPlayers.Length;
	enemyData.health = bonus + bonus2;
	enemyHealth.text = enemyData.health.ToString();
}
```

This is currently only used in the endless loop mode to continually increase the enemy health.

###TakeDamage();

```csharp
public void TakeDamage(int x)
{
	this.photonView.RPC("TakeDamageRPC", RpcTarget.All, x);
}
```

This method is used to call ['TakeDamageRPC()'](#enemy-controller-pun-methods-takedamagerpc) and makes the enemy take damagea accross all clients at the same time.

###UpdateHealth();

```csharp
//Updates the enemy health, and handles win condition
public void Updatehealth(int x)
{
	//If the enemy has no health left and the player won
	if(x <= 0)
	{
		//Disable main canvas and enable victory screen
		if(PhotonNetwork.IsMasterClient)
		{
			NetworkController network = GameObject.Find("NetworkController").GetComponent<NetworkController>();
			network.ResetGame();
		}
		controller.SetIsRunning(false);
		canvas.SetActive(false);
		death.SetActive(false);
	}
	else
	{	
		//Update health and health display
		enemyData.health = x;
		enemyHealth.text = enemyData.health.ToString();
	}
}
```

This method is used to update thee enemy health.  Crucially, it also checks to make sure that the enemy isn't dead, and then updates ['enemyHealth'](#enemy-controller-private-variables-enemyHealth).

##Private Methods

###EnterLevel();

```csharp
private void EnterLevel () 
{	
	//Get controller
	controller = GetGameController();
	//Get canvas
	canvas = GetGameCanvas();
	//Get local player
	player = GetPlayer();
	//Get health bar animator
	healthBar = GetHealthBarAnimator();
	//Set various UI text displays
	enemyHealth = GetHealthText();
	enemyNameText = GetNameText();
	enemyDamageDisplay = GetDamageText();
	//Update with starting health value
	Updatehealth(enemyData.health);
	UpdateShields(enemyData.shields);
	UpdateName(enemyData.enemyName);
	SetNextEnemyAttack();
}
```

This is called whenever our level is first initialized so that our object lookups don't go towards inactive objects.  It assigns a lot of our public variables, updates the enemy data properly, and picks the next enemy attack.

###GetHealthBarAnimator();

```csharp
private Animator GetHealthBarAnimator()
{
	Animator anim = GameObject.Find("enemyHealthBarAnimator").GetComponent<Animator> ();
	return anim;
}
```

This is used to find the animator of our enemy health bar and return it.

