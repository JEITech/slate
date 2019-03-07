#Class Controller

The ClassController is used to hold data on the players currently selected class, including the name of the class and all of the class cards.

##Public Variables

###className

```csharp
public string className;
```

This string is used to hold the name of the current class.

###classCards

```csharp
public List<CardData> classCards;
```

This is a list of all cards related to the selected class.

###specialCards

```csharp
public List<CardData> specialCards;
```

This is a list of all class cards that are also flagged as requiring special processing.

###starterCards

```csharp
public List<CardData> starterCards;
```

This is a list of all class cards that should be in the players deck from the start of the game.

##Photon

###OnPhotonSerializeView(PhotonStream stream, PhotonMessageInfo info);

```csharp 
public void OnPhotonSerializeView(PhotonStream stream, PhotonMessageInfo info)
{
	
}
```

This method does nothing.  However, it is required for us to implement this method on any networked game objects, so here it is, in all its glory.

##Public Methods

###GetAllClassCards();

```csharp
public List<CardData> GetAllClassCards()
	{
		return classCards;
	}
```

This method simply returns the [`classCards`](#class-controller-public-variables-classcards) variable.
