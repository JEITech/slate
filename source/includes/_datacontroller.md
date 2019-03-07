#Data Controller

DataController is a class used to read from JSON files andreturn insantiated C# classes from the data found within those files.

##Public Variables

###cards

```csharp
public List<CardData> cards;
```
This variable is used to hold deck information.  It gets returned by the DataController via the method [`GetDeck()`](#data-controller-public-methods-getdeck).

###classData

```csharp
public ClassData classData;
```

This variable is used to hold class information.  It gets returned by the DataController via the method [`GetClassData(string className)`](#data-controller-public-methods-getclassdata-string-classname).

###cardDataPrefix

```csharp
public string cardDataPrefix;
```

This variable is used to hold the prefix of the directory where our card data JSON files are stored.  It gets combined with in the method [`GetClassData(string className)`](#data-controller-public-methods-getclassdata-string-classname) to determine the proper path to our card data files.


###playerDataPrefix

```csharp
public string playerDataPrefix;
```

This variable is used to hold the prefix of the directory where our player data JSON files are stored.  It gets combined with the default streaming assets path in the method [`LoadPlayerData()`](#data-controller-private-methods-loadplayerdata) to determine the proper path to our player data files.


###enemyDataPrefix

```csharp
public string enemyDataPrefix;
```

This variable is used to hold the prefix of the directory where our enemy data JSON files are stored.  It gets combined with the default streaming assets path in the method [`LoadEnemyData(string enemyName)`](#data-controller-private-methods-loadenemydata-string-enemyname) to determine the proper path to our player data files.



###classDataPrefix

```csharp
public string classDataPrefix;
```

This variable is used to hold the prefix of the directory where our class data JSON files are stored.  It gets combined with the class name argument in [`GetClassData(string className)`](#data-controller-public-methods-getclassdata-string-classname) and then used in [`LoadClassData()`](#data-controller-private-methods-loadclassdata), where it is combined with the default streaming assets path.

###storyDataPrefix

```csharp
public string storyDataPrefix;
```

This varibale is used to hold the prefix of the directory where our story data JSON files are stored.  It gets used in the method [`GetStoryData('string storyName)`](#data-controller-public-methods-getstorydata-string-storyname), where it is combined with the `storyName` argument to call the [`LoadStoryData(string storyName)`](#data-controller-private-methods-loadstorydata-string-storyname) method and then return the relevant story data.


##Unity Default Methods

###Start();

```csharp
void Start () 
{
	DontDestroyOnLoad (gameObject);
	SceneManager.LoadScene ("SplashScreen");	
}
```

On start, the DataController marks itself as to not be destroyed on a scene switch, and then loads the splash screen.

##Public Methods

###GetClassData(string className);

```csharp
public ClassData GetClassData(string className)
{
	string classDataPath = classDataPrefix + className;
	classData = LoadClassData(classDataPath);
	return classData;
}
```

This method is used to load all relevant class data.  It accepts the string `className` as an argument, and combines that with the [`classDataPrefix`](#data-controller-public-variables-classdataprefix) to determine the file path to the relevant JSON files for the selected class.  It then loads that class data with the string, and returns said data.

###GetPlayerData();

```csharp
public PlayerData GetPlayerData()
{
	PlayerData player = LoadPlayerData();
	return player;
}
```

This method is used simply to load and then return the `player` variable. Currently it only get's called once at the start of the game to get player starter data, though could be modified to handle save games.

###GetStoryData(string storyName);

```csharp
public StoryData GetStoryData(string storyName)
{
	StoryData storyData = LoadStoryData(storyName);
	return storyData;
}
```

This method is used to load all relevant story data.  It accepts the string `storyName` as an argument, and sends that off through the [`LoadStoryData(string storyName)`](#data-controller-private-methods-loadstorydata-string-storyname) method in order to load the relevant story data.  It then returns the data. 

###GetDeck();

```csharp
public List<CardData> GetDeck()
{
  LoadCardData();
	return cards;
}
```

This method is used simply to load default card data and return the [`cards`](#data-controller-public-variables-cards) variable.

###GetEnemyData(string enemyName);

```csharp
public EnemyData GetEnemyData(string enemyName)
{
	EnemyData enemy = LoadEnemyData(enemyName);
	return enemy;
}
```

This method is used simply to load and then return enemy data based off of the enemy's name, `enemyName`.  It calls [`LoadEnemyData(string enemyName)`](#data-controller-private-methods-loadenemydata-string-enemyname), passing the enemy's name, and then returns the `enemy` variable.

##Private Methods

###LoadClassData();

```csharp
private ClassData LoadClassData(string path)
{
	ClassData classData = new ClassData();
	//Read the classData.json file and create ClassData from it
	string filepath = Path.Combine(Application.streamingAssetsPath, path);
	DirectoryInfo dir = new DirectoryInfo (filepath);
	string jsonData = File.ReadAllText(Path.Combine(filepath, "classData.json"));
	classData = JsonUtility.FromJson<ClassData>(jsonData);
	//Look through all the card files and create CardData lists from them
	string classPath = path + "/cards";
	filepath = Path.Combine(Application.streamingAssetsPath, classPath);
	dir = new DirectoryInfo (filepath);
	FileInfo[] cardFiles  = dir.GetFiles("*.json");
	for(int i = 0 ; i < cardFiles.Length; i++)
	{
		//Ignore Unity meta files
		if(cardFiles[i].Extension != ".meta")
		{
			//Add card to the master list of cards, and the current instance of ClassData
			jsonData = File.ReadAllText(cardFiles[i].FullName);
			CardData cardToAdd = JsonUtility.FromJson<CardData>(jsonData);
			classData.cards.Add(cardToAdd);
			//Add special cards and starter cards to unique lists
			if(cardToAdd.isSpecial)
			{
				classData.specialCards.Add(cardToAdd);
			}
			if(cardToAdd.isStarter)
			{
				classData.starterCards.Add(cardToAdd.starterData);
			}
		}
	}
	return classData;
}
```

This method is used to access the JSON files for a specific class that a player has selected.  It gets all the data pertaining to a class, including cards, bonus effects, artwork, and anything else we need.  Because cards are all represented as their own JSON file, we must loop through all cards found in the class sub-directory and add those cards to the `classData` variable before returning it.

###LoadCardData();

```csharp
private void LoadCardData()
{
	//Read through all card files
	string filepath = Path.Combine(Application.streamingAssetsPath, cardDataPrefix);
	DirectoryInfo dir = new DirectoryInfo (filepath);
	FileInfo[] cardFiles = dir.GetFiles("*.json*");
	int count = dir.GetFiles().Length;
	for(int i = 0; i < count; i++)
	{	
		//Ignore Unity meta files
		if(cardFiles[i].Extension != ".meta")
		{
			//Add card to the master list of cards
			string jsonData = File.ReadAllText(cardFiles[i].FullName);
			cards.Add(JsonUtility.FromJson<CardData>(jsonData));
		}
	}
}
```

This method is used to access the JSON files for all of our basic starter cards.  It reads the cards based off of the [`cardDataPrefix`](#data-controller-public-variables-carddataprefix) variable, and then adds all found card data to the [`cards`](#data-controller-public-variables-cards) variable.

###LoadStoryData(string storyName);

```csharp
private StoryData LoadStoryData(string storyName)
{
	StoryData storyData = new StoryData();
	//Read through all story files
	string filepath = Path.Combine(Application.streamingAssetsPath, storyDataPrefix);
	DirectoryInfo dir = new DirectoryInfo (filepath);
	string storyPath = storyName + ".json";
	FileInfo[] fileInfo = dir.GetFiles(storyPath);
	string jsonData = File.ReadAllText(fileInfo[0].FullName);
	storyData = JsonUtility.FromJson<StoryData>(jsonData);
	return storyData;
}
```

This method is used to access the JSON file for a specific story moment.  It reads the file based off the [`storyDataPrefix`](#data-controller-public-variables-storydataprefix) variable, and then finds the corresponding JSON file with the `storyName` argument.  Once it has the data, it uses it to set `storyData` and return it.

###LoadPlayerData();

```csharp
private PlayerData LoadPlayerData()
{
	PlayerData playerData = new PlayerData();
	//Read the playerData.json file and create PlayerData from it
	string filepath = Path.Combine(Application.streamingAssetsPath, playerDataPrefix);
	string jsonData = File.ReadAllText(Path.Combine(filepath, "playerData.json"));
	playerData = JsonUtility.FromJson<PlayerData>(jsonData);
	//Read through all deck data files
	filepath = Path.Combine(Application.streamingAssetsPath, playerDataPrefix);
	DirectoryInfo dir = new DirectoryInfo (Path.Combine(filepath, "deckData"));
	FileInfo[] deckFile = dir.GetFiles("*.json*");
	int count = dir.GetFiles().Length;
	for(int i = 0; i < count; i++)
	{
		//Ignore Unity meta files
		if(deckFile[i].Extension != ".meta")
		{
			//Create an instance of DeckBuilder for each deck data file in the players directory
			jsonData = File.ReadAllText(deckFile[i].FullName);
			playerData.deck.Add(JsonUtility.FromJson<DeckBuilder>(jsonData));
		}
	}
	//Load class specific starter cards into newly created player deck
	foreach(DeckBuilder card in classData.starterCards)
	{
		playerData.deck.Add(card);
	}	
	return playerData;
}
```

This method is used to access the JSON files for our Player.  It could support save game states but currently is just called at the start of the game to get the default starting values.  It loads the player data using the [`playerDataPrefix`](#data-controller-public-variables-playerdataprefix) varibale, and uses it to set and return the `playerData` variable.

###LoadEnemyData(string enemyName);

```csharp
private EnemyData LoadEnemyData(string enemyName)
{
	EnemyData enemyData = new EnemyData();
	//Read the enemyData.json file land create EnemyData from it
	string filepath = Path.Combine(Application.streamingAssetsPath, enemyDataPrefix);
	filepath = Path.Combine(filepath, enemyName);
	string jsonData = File.ReadAllText(Path.Combine(filepath, "enemyData.json"));
	enemyData = JsonUtility.FromJson<EnemyData>(jsonData);
	//Read through all enemy attacks
	DirectoryInfo dir = new DirectoryInfo (Path.Combine(filepath, "attacks"));
	FileInfo[] deckFile = dir.GetFiles("*.json*");
	int count = dir.GetFiles().Length;
	for(int i = 0; i < count; i++)
	{
		//Ignore Unity meta files
		if(deckFile[i].Extension != ".meta")
		{
			//Add attack to list of EnemyAttackData
			jsonData = File.ReadAllText(deckFile[i].FullName);
			enemyData.attacks.Add(JsonUtility.FromJson<EnemyAttackData>(jsonData));
		}
	}
	return enemyData;
}
```

This method is used to access the JSON files for a specific enemy based off the enemy name.  It uses the [`enemyDataPrefix`](#data-controller-public-variables-enemydataprefix) combined with the argument `enemyName` to load all related data, and sets all of the values inside of the `enemyData` variable before returning it.