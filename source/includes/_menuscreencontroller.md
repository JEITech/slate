#Menu Screen Controller

The MenuScreenController is used for anything we need to do on the main menu.

##Public Methods

###StartMultiplayer();

```csharp 
public void StartMultiplayer()
{
	DisableGameObjects();
	EnableButton();
}
```

This method is used to make sure that for right now, only the multiplayer button shows up on the main menu.  It calls [`DisableGameObjects()`](#menu-screen-controller-private-methods-disablegameobjects) and [`DisableButton`](#menu-screen-controller-private-methods-disablebutton).

##Private Methods

###DisableGameObjects();

```csharp
private void DisableGameObjects()
{
	GameObject.Find("single").SetActive(false);
	GameObject.Find("multi").SetActive(false)
	GameObject.Find("teehee").SetActive(false);
}
```

This method is used to disable the canvas items for the other gameplay modes for now.

###DisableButton();

```csharp
private void EnableButton()
{
	GameObject.Find("Connect Button").GetComponent<Button> ().interactable = true;
}
```

This method is used to make the "Start game" button interactable once we are connected to the photon network.