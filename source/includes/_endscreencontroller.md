#End Screen Controller

The EndScreenController is used to handle the game state one the boss in the current scene has died.

##Public Methods

###LeaveRoom()

```csharp
public void LeaveRoom()
{
    PhotonNetwork.LeaveRoom();
}
```

This method can be called at any time to take the player out of the current lobby and send them back to the main menu.