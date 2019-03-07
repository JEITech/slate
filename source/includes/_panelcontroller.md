#Panel Controller

Currently this class is only used to slide the class select interface up and down in the lobby.

##Public Variables

###classSelect

```csharp
public GameObject classSelect;
```

This variable hold a reference to the class select dropdown parent game object.

##Public Methods

###EnableClassSelect();

```csharp
public void EnableClassSelect()
{
	Animator playerAnim = classSelect.GetComponent<Animator> ();
	playerAnim.SetBool("Picking", true);
}
```

This method is used slide the class select panel onto the screen by updating its animator.


###DisableClassSelect();

```csharp
public void DisableClassSelect()
{
	Animator playerAnim = classSelect.GetComponent<Animator> ();
	playerAnim.SetBool("Picking", false);
}
```

This method is used to slide the class select panel off of the screen by updating its animator.