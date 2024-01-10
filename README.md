# UE-Cheat-Sheet
my lessons learned.  
[Alex Forsythy repo](https://github.com/awforsythe/Repsi/tree/main)  
[Ari](https://ari.games/), [Ari's Notion](https://flassari.notion.site/Ari-s-Unreal-Engine-Notes-1a75e43f4014464984d4fae0617e5cef)

* lighter version of GetAllActorsOfClass()
```cpp
for (AIController* Controller : TActorRange<AShooterAIController>(GetWorld()))
```
  
* HTF do I see changes made in the editor?
```cpp
// .h
UPROPERTY(EdityAnywhere, BlueprintReadWrite)
float Radius{500.f};
virtual void PostEditChangeProperty(FpropertyChangedEvent& PropertyChangedEvent) override;
// attach SomeOtherClass
// .cpp
void MyClass::PostEditChangeProperty(FpropertyChangedEvent& PropertyChangedEvent)
{
  Super::PostEditChangeProperty(&PropertyChangedEvent) // ?that & could be Rider
  SomeOtherClass->SetRadius(Radius);
}
```

* HTF do I process OnDestroyed while in editor?
```cpp
// .h
virtural void OnComponentDestroyed(bool bDestroyingHierarchy) override;
// attach SomeOtherClass
// .cpp
void MyClass::OnComponentDestroyed(bool bDestroyingHierarchy)
{
  Super::OnComponentDestroyed(bool bDestroyingHierarchy);
  SomeOtherClass->DestroyComponent();
}
```


## Replication  
- Be shure to pass in parameters to RPCs and not use member vars in the body, else RPCs don't work.  
- Unless PlayerCharacter/Pawn/player related code, HasAuthority().  else: Pawn->IsLocallyControlled(), Controller->IsLocalController(), !IsDedicatedServer()  
- MulticastRPC doesn't replicate if client doesn't have open channel.  
- Replicated [x] is cheaper than RPC.  Try and avoid RPC when possible.

### HTF do I
* have a client update the color of a material on an Actor and all clients & server see the change?
1. PC calls PC.ServerRPCSetSomeActorColor(Color, SomeActor).  // that's the way clients talk to the server.
1. Which calls SomeActor.SetColor(Color) w/ Notify or SomeActor->Color = aColor.   
1. Which calls SomeActor.OnRep_Color() // LinearColor Color is a public member var
1. OnRep_Color() sets the Dynamic_MI.ParameterName  // that's one way the server talks to clients.  lazy comprared to RPC.  if behind HasAuth() then it doesn't run on clients
Server & clients will see the change, eventually.

### Notes until I make my own

![Screenshot 2023-12-12 104409](https://github.com/CodyMcCarty/UE-Cheat-Sheet/assets/68304541/6ecd0095-7225-4305-9ce5-86f13f1d6b74)  

![Screenshot 2023-12-12 104429](https://github.com/CodyMcCarty/UE-Cheat-Sheet/assets/68304541/14e62dd5-91d0-4b61-babf-00578786c180)  

![Screenshot 2023-12-12 104452](https://github.com/CodyMcCarty/UE-Cheat-Sheet/assets/68304541/eb81215e-f76e-46c8-a63e-49f4be217529)  

![Screenshot 2023-12-12 104515](https://github.com/CodyMcCarty/UE-Cheat-Sheet/assets/68304541/b51a5b54-4581-4cce-811e-4f2467fbb88d)  

// todo update this as I use it more  
| Runs On | Class | Use Case |
| ----------- | ----------- | ----------- |
| Server-Only | GameMode | u |
| Server-Only | GameSession | u |
| Replicated to all clients | GameState | u |
| Replicated to all clients | PlayerState | u |
| Replicated to relevant clients | Pawn | u |
| Replicated to Owning Client | PlayerController | u |
