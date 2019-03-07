---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - csharp : C#

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

#Introduction

Documentation for Scribes Of Fate C# stuff.  Woo pogchamp.

#Card Controller

CardController is a class placed on each instance of a card in the scene to allow it to be controlled and applied within the game.  It handles dragging and dropping cards, as well as applying the effects of the card (damage, shields, etc.)

##Public Variables

#Data Controller

DataController is a class used to read from JSON files andreturn insantiated C# classes from the data found within those files.

##Public Variables

###cards

```csharp
public List<CardData> cards;
```
This variable is used to hold deck information, as it is a list of [`CardData`](#carddatatype) type objects.  It gets returned by the DataController via the method [`GetDeck()`](#getdeck).

###player

```csharp
public PlayerData player;
```

This variable is used to hold player information, and is of the type [`PlayerData`](#playerdatatype).  It gets returned by the DataController via the method [`GetPlayerData()`](#getplayerdata).

###enemy

```csharp
public EnemyData enemy;
```

This variable is used to hold enemy information, and is of the type [`EnemyData`](#enemydatatype).  It gets returned by the DataController via the method [`GetEnemyData(string enemyName)`](#getenemydata-string-enemyname).

###classData

```csharp
public ClassData classData;
```

This variable is used to hold class information, and is of the type [`ClassData`](#classdatatype).  It gets returned by the DataController via the method [`GetClassData(string className)`](#getclassdata-string-classname).

###storyData

```csharp
public StoryData storyData;
```

This variable is used to hold information for the 'Choose your own adevnture' style story segments of the game, and is of the type [`StoryData`](#storydatatype).  It gets returned by the DataController via the method [`GetStoryData(string storyName)`](#getstorydata).

###classPath

```csharp
public string classPath;
```

This variable holds the name of the players currently selected class.  This is used in combination with [`classDataPrefix`](#classdataprefix) to determine the file path of the JSON files for the currently selected class via the method [`GetClassData(string className)`](#getclassdata-string-classname).


###cardDataPrefix

```csharp
public string cardDataPrefix;
```

This variable is used to hold the prefix of the directory where our card data JSON files are stored.  It gets combined with [`classPath`](#classpath) in the method [`GetClassData(string className)`](#getclassdata-string-classname) to determine the proper path to our card data files.


###playerDataPrefix

```csharp
public string playerDataPrefix;
```

This variable is used to hold the prefix of the directory where our player data JSON files are stored.  It gets combined with the default streaming assets path in the method [`LoadPlayerData()`](#loadplayerdata) to determine the proper path to our player data files.


###enemyDataPrefix

```csharp
public string enemyDataPrefix;
```

This variable is used to hold the prefix of the directory where our enemy data JSON files are stored.  It gets combined with the default streaming assets path in the method [`LoadEnemyData(string enemyName)`](#loadenemydata-string-enemyname) to determine the proper path to our player data files.



###classDataPrefix

```csharp
public string classDataPrefix;
```

This variable is used to hold the prefix of the directory where our class data JSON files are stored.  It gets combined with [`classPath`](#classpath) in [`GetClassData(string className)`](#getclassdata-string-classname) and then used in [`LoadClassData()`](#loadclassdata), where it is combined with the default streaming assets path.

###storyDataPrefix

```csharp
public string storyDataPrefix;
```

This varibale is used to hold the prefix of the directory where our story data JSON files are stored.  It gets used in the method [`GetStoryData('string storyName)`](#getstorydata-string-storyname), where it is combined with the `storyName` argument to call the [`LoadStoryData(string storyName)`](#loadstorydata-string-storyname) method and then return the [`storyData`](#storydata) variable.


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
	classPath = className;
	classDataPrefix = "classes/" + classPath;
	LoadClassData();
	LoadPlayerData();
	LoadCardData();
	return classData;
}
```

This method is used to load all relevant class data.  It accepts the string `className` as an argument, and combines that with the [`classDataPrefix`](#classdataprefix) to determine the file path to the relevant JSON files for the selected class.  It then loads that class data, as well as player data and card data.  Lastly, it returns the [`classData`](#classdata) variable.

###GetPlayerData();

```csharp
public PlayerData GetPlayerData()
{
	return player;
}
```

This method is used simply to return the [`player`](#player) variable. 

###GetStoryData(string storyName);

```csharp
public StoryData GetStoryData(string storyName)
{
	LoadStoryData(storyName);
	return storyData;
}
```

This method is used to load all relevant story data.  It accepts the string `storyName` as an argument, and sends that off through the [`LoadStoryData(string storyName)`](#loadstorydata-string-storyname) metho in order to load the relevant story data.  It then returns the variable [`storyData`](#storydata). 

###GetDeck();

```csharp
public List<CardData> GetDeck()
{
	return cards;
}
```

This method is used simply to return the [`cards`](#cards) variable.

###GetEnemyData(string enemyName);

```csharp
public EnemyData GetEnemyData(string enemyName)
{
	LoadEnemyData(enemyName);
	return enemy;
}
```

This method is used simply to load and then return enemy data based off of the enemy's name, [`enemyName`](#enemyname).  It calls [`LoadEnemyData(string enemyName)`](#loadenemydata), passing the enemy's name, and then returns the [`enemy`](#enemy) variable.

##Private Methods

###LoadClassData();

```csharp
private void LoadClassData()
{
	//Read the classData.json file and create ClassData from it
	string filepath = Path.Combine(Application.streamingAssetsPath, classDataPrefix);
	DirectoryInfo dir = new DirectoryInfo (filepath);
	string jsonData = File.ReadAllText(Path.Combine(filepath, "classData.json"));
	classData = JsonUtility.FromJson<ClassData>(jsonData);
	//Look through all the card files and create CardData lists from them
	string classPath = classDataPrefix + "/cards";
	filepath = Path.Combine(Application.streamingAssetsPath, classPath);
	dir = new DirectoryInfo (filepath);
	Debug.Log(classPath);
	FileInfo[] cardFiles  = dir.GetFiles("*.json");
	for(int i = 0 ; i < cardFiles.Length; i++)
	{
		//Ignore Unity meta files
		if(cardFiles[i].Extension != ".meta")
		{
			//Add card to the master list of cards, and the current instance of ClassData
			jsonData = File.ReadAllText(cardFiles[i].FullName);
			CardData cardToAdd = JsonUtility.FromJson<CardData>(jsonData);
			cards.Add(cardToAdd);
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
}
```

This method is used to access the JSON files for a specific class that a player has selected.  It gets all the data pertaining to a class, including cards, bonus effects, artwork, and anything else we need.  Because cards are all represented as their own JSON file, we must loop through all cards found in the class sub-directory and add those cards to the [`classData`](#classdata) variable.

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

This method is used to access the JSON files for all of our basic starter cards.  It reads the cards based off of the [`cardDataPrefix`](#carddataprefix) variable, and then adds all found card data to the [`cards`](#cards) variable.

###LoadStoryData(string storyName);

```csharp
private void LoadStoryData(string storyName)
{
	//Read through all story files
	string filepath = Path.Combine(Application.streamingAssetsPath, storyDataPrefix);
	DirectoryInfo dir = new DirectoryInfo (filepath);
	string storyPath = storyName + ".json";
	FileInfo[] fileInfo = dir.GetFiles(storyPath);
	string jsonData = File.ReadAllText(fileInfo[0].FullName);
	storyData = JsonUtility.FromJson<StoryData>(jsonData);
}
```

This method is used to access the JSON file for a specific story moment.  It reads the file based off the [`storyDataPrefix`](#storydataprefix) variable, and then finds the corresponding JSON file with the `storyName` argument.  Once it has the data, it uses it to set [`storyData`](#storydata).

###LoadPlayerData();

```csharp
private void LoadPlayerData()
{
	//Read the playerData.json file and create PlayerData from it
	string filepath = Path.Combine(Application.streamingAssetsPath, playerDataPrefix);
	string jsonData = File.ReadAllText(Path.Combine(filepath, "playerData.json"));
	player = JsonUtility.FromJson<PlayerData>(jsonData);
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
			player.deck.Add(JsonUtility.FromJson<DeckBuilder>(jsonData));
		}
	}
	//Load class specific starter cards into newly created player deck
	foreach(DeckBuilder card in classData.starterCards)
	{
		player.deck.Add(card);
	}	
}
```

This method is used to access the JSON files for our Player.  It could support save game states but currently is just called at the start of the game to get the default starting values.  It loads the player data using the [`playerDataPrefix`](#playerdataprefix) varibale, and sets all of the values inside of the [`player`](#player) variable.

###LoadEnemyData(string enemyName);

```csharp
private void LoadEnemyData(string enemyName)
{
	//Read the enemyData.json file land create EnemyData from it
	string filepath = Path.Combine(Application.streamingAssetsPath, enemyDataPrefix);
	filepath = Path.Combine(filepath, enemyName);
	string jsonData = File.ReadAllText(Path.Combine(filepath, "enemyData.json"));
	enemy = JsonUtility.FromJson<EnemyData>(jsonData);
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
			enemy.attacks.Add(JsonUtility.FromJson<EnemyAttackData>(jsonData));
		}
	}
}
```

This method is used to access the JSON files for a specific enemy based off the enemy name.  It uses the [`enemyDataPrefix`](#enemydataprefix) combined with the argument `enemyName` to load all related data, and sets all of the values inside of the [`enemy`](#enemy) variable.

