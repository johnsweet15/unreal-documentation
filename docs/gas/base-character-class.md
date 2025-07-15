# Base Character Class

## Overview
This will be base class that will share functionality across the main character and enemy characters.

## Creating a base class
To create a class in the editor, go to `Tools > New C++ Class...` and select the parent class.  For this example, we will choose the Character class.  You can select `Public` if you want your header files to be in a `Public` folder and `.cpp` files to be in a `Private` folder.  We are going to name the class `AuraCharacterBase` and create it inside a `Character` folder.

Since this is a base class, we can make the class abstract in the header file, meaning it cannot be dragged into a level.  We can also remove the `Tick` function since tick won't be need on a base class and the `SetupPlayerInputComponent` function since player input will be setup on the player character class.

``` cpp title="Public/Character/AuraCharacterBase.h"
UCLASS(Abstract)
class AURA_API AAuraCharacterBase : public ACharacter
{
	GENERATED_BODY()

public:
	AAuraCharacterBase();

protected:
	virtual void BeginPlay() override;

};
```
``` cpp title="Private/Character/AuraCharacterBase.cpp"
#include "Character/AuraCharacterBase.h"

AAuraCharacterBase::AAuraCharacterBase()
{
	PrimaryActorTick.bCanEverTick = false; // tick not needed
}

void AAuraCharacterBase::BeginPlay()
{
	Super::BeginPlay();
	
}
```

## Create a child class
Find your base class in the Unreal editor file explorer, right click and select `Create C++ class derived from AuraCharacterBase`.  Do this and create a class `AuraCharacter` and `AuraEnemy` in the `Character` folder.

You should see that these classes derive from `AuraCharacterBase`:

``` cpp title="Public/Character/AuraCharacter.h"
UCLASS()
class AURA_API AAuraCharacter : public AAuraCharacterBase
{
	GENERATED_BODY()

public:
  AAuraCharacter();
	
};
```

## Adding a skeletal mesh and socket to the base class
### Define the skeletal mesh in the header file
We want both the character and enemy classes to have weapons, so we will add a wepaon socket to the base class:
``` cpp title="Public/Character/AuraCharacterBase.h" linenums="1" hl_lines="4 5"
protected:
    virtual void BeginPlay() override;

    UPROPERTY(EditAnywhere, Category = "Combat")
    TObjectPtr<USkeletalMeshComponent> Weapon;
```
!!! note "A note about `TObjectPtr` and `UPROPERTY`"
    `TObjectPtr` behaves the same as a raw pointer but with some additional features:

    * Access tracking: We can track how often a pointer is accessed or dereferenced
    * Optional lazy load: An asset can not be loaded until its used

    `UPROPERTY` exposes the component to the Unreal Editor under the `Combat` category and `EditAnywhere` means it can be edited from anywhere within the Editor.

### Create the skeletal mesh in the .cpp file

``` cpp title="Private/Character/AuraCharacterBase.cpp" linenums="1" hl_lines="5-7"
AAuraCharacterBase::AAuraCharacterBase()
{
	PrimaryActorTick.bCanEverTick = false;

	Weapon = CreateDefaultSubobject<USkeletalMeshComponent>("Weapon");
	Weapon->SetupAttachment(GetMesh(), FName("WeaponHandSocket"));
	Weapon->SetCollisionEnabled(ECollisionEnabled::NoCollision);

}
```

* Line 5: Create the Weapon object using a skeletal mesh that will be set in the editor
* Line 6: Attach the weapon to `GetMesh()` (function inherited from `ACharacter` to get the character mesh) at the socket name `WeaponHandSocket` (doesn't exist on the character mesh yet)
* Line 7: Disable collision for the weapon

We will create the socket and attach it to the character mesh in the next section.

## Create a blueprint class
Start by going to the editor and creating a folder `Content/Blueprints/Character/Aura`.  Then right click and select `Blueprint Class` and search for `AuraCharacter`.  Select it and call it `BP_AuraCharacter`.  This will create an instance of `AuraCharacter`.  Set the character skeletal mesh and relocate, resize, rotate as needed.

### Create the weapon socket
Open the character skeletal mesh, find the left hand bone, right click and select `Add Socket`.  Make sure to rename the socket to `WeaponHandSocket`.  Right click again and select `Add Preview Asset` and select your weapon.  Then on the right `Preview Scene` panel select `Preview Controller` and click `Use Specific Animation`, then select an animation that holds the weapon.  Move and rotate the socket as needed.

Go back to the character blueprint and set the weapon skeletal mesh.  It should be located in the correct spot with the correct rotation.