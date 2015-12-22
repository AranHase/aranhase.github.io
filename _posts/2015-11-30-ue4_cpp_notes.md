---
layout: post
title: Unreal Engine 4 and C++ Notes
tags: gamedev, cpp, UE4
---

A list of notes and snippets for Unreal Engine 4 and C++.
My personal cheatsheet :)

clang-format
------------

Visual Studio 2015 is incapable of formatting UE4 code properly because of the macros.
[`clang-format`][1] can format the code properly, but will mess with the order of your
includes. The problem is, UE4 **requires** that the `<filename>.generated.h` include file
to be the last include. So, place this `.clang-format` file in the
`<Game Folder>/Source` directory:


{% highlight python %}
BasedOnStyle: LLVM
IndentWidth: 2
IncludeCategories:
  - Regex:           '.*(generated)'
    Priority:        10
  - Regex:           '.*'
    Priority:        1
{% endhighlight %}

The `IncludeCategories` entry is the important part. It tells the order of the include.
The `<filename>.generated.h` file will always be placed last because it will have a
higher priority number.

Some macros used by UE4 cannot have break lines. So, disable clang-format around these
macros enclosing them like this:


{% highlight cpp %}
// clang-format off
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FTowerDataStatsChanged, UBaseTowerData *, TowerData);
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FTowerDataUpgraded, UBaseTowerData *, TowerData);
// clang-format on
{% endhighlight %}


Visual Studio is bad?
--------------------

![vs-slow][2]

Is it familiar to you?.. it was too me :(

Solved it the extreme way by combining vim + emacs.
[`Spacemacs`][3] to the rescue...

Here is my modifications to the `.spacemacs` file so far:

{% highlight lisp %}
;; Enable the C-C++ layer, but open all the headers, even ".c",
;; in C++-mode.
(defun dotspacemacs/layers ()
  (setq-default
   dotspacemacs-configuration-layers
   '(
     (c-c++ :variables
            c-c++-default-mode-for-headers 'c++-mode)
     )
  )
)

(defun dotspacemacs/user-config ()
  ;; Bind <TAB> to "clang-format" when in C++ mode
  ;; (works on selections)
  (add-hook 'c++-mode-hook 'clang-format-bindings)
    (defun clang-format-bindings ()
      (define-key c++-mode-map [tab] 'clang-format-buffer))

  ;; Show line numbers by default
  (global-linum-mode)
)
{% endhighlight %}

I keep both Visual Studio and Spacemacs open when coding. Spacemacs is useful
when editing files, and when I change a file, it is automatically updated in Visual
Studio. If I ever need to use Intellisense or something that Visual Studio has,
then I can just switch windows and use it. Also, compilations and debug are handled
inside Visual Studio too.

### Speeding up Visual Studio

Visual Studio with [VsVim](https://github.com/jaredpar/VsVim), clang-format,
[RelativeLineNumbers](https://visualstudiogallery.msdn.microsoft.com/74d68e2b-ff64-4c51-a2ed-d8b164eee858)
and [Visual Assist](http://www.wholetomato.com/default.asp) (Paid plugin). Also,
[Hack](https://github.com/chrissimpkins/Hack) font is cool.

Print To Screen
---------------

{% highlight cpp %}
// inside your top header
#define PRINTSCR(text) if (GEngine) GEngine->AddOnScreenDebugMessage(-1, 3, FColor::White, text)
{% endhighlight %}

Log
---

{% highlight cpp %}
// UE_Log(<Category>, <Level>, <Message>, <Message Params>);
UE_LOG(LogTemp, Warning, TEXT("It is %f percent full. Answer: %d"), 0.5f, 42);
{% endhighlight %}

Levels:

 * **Log**: will print in grey
 * **Warning**: will print in yellow
 * **Error**: will print in red
 * **Fatal**: this will force a crash with its message when executed

### Create your own log category

Log categories can be used to easier differentiate between types of logs.

{% highlight cpp %}
// in .h
// DECLARE_LOG_CATEGORY_EXTERN(<Category Name>, <Default Verbosity>, <Compile Time Verbosity>);
DECLARE_LOG_CATEGORY_EXTERN(LogGameplay, Log, All);

// in .cpp
DEFINE_LOG_CATEGORY(LogGameplay);
{% endhighlight %}

Full Details [here](https://wiki.unrealengine.com/Logs,_Printing_Messages_To_Yourself_During_Runtime) (Thanks Rama <3)

enum
----

{% highlight cpp %}
UENUM(BlueprintType)
enum class ETowerType : uint8 {
  VE_Solar UMETA(DisplayName = "Solar"),
  VE_Cube UMETA(DisplayName = "Cube"),
  VE_Battery UMETA(DisplayName = "Battery")
};

// usage
Type = ETowerType::VE_Cube;
{% endhighlight %}


Direct link to UPROPERTY, UFUNCTION, UCLASS properties
-------

 * [UPROPERTY, Properties specifiers][4]
 * [UFUNCTION, Functions specifiers][5]
 * [UCLASS, Class specifiers][6]

What the `T` in `TArray` means? `F` in `FText`? etc...
------------

Classes starting with:

 * `T` means they are templated classes
 * `U` means they are derived from `UObject`
 * `A` means they are derived from `AActor`
 * `I` are interfaces
 * `F` are everything else?

Pure Virtual functions
--------

Unreal requires that every class must be instantiable, so you can't
declare pure virtual functions. But, Unreal provides the `PURE_VIRTUAL` 
macro to help. Check its implementation:


{% highlight cpp %}
#define PURE_VIRTUAL(func,extra) { LowLevelFatalError(TEXT("Pure virtual not implemented (%s)"), TEXT(#func)); extra }
{% endhighlight %}

It requires two parameters. The first one should be the name of your function so it is easier to debug.
The second one is some extra code placed inside the function the macro is creating. Use it to return
a compatible value with your function return type. Check the example:


{% highlight cpp %}
UCLASS()
class UFoo : UObject {
  GENERATED_BODY()

public:
  virtual void RefreshDamage() PURE_VIRTUAL(UFoo::RefreshDamage, );
  
  virtual bool IsOkay() PURE_VIRTUAL(UFoo::IsOkay, return false);
}

UCLASS()
class UBar : UFoo {
  GENERATED_BODY()

public:
  void RefreshDamage() override; // use the "override" keyword
  bool IsOkay() override;
}
{% endhighlight %}



Functions != Methods
---------------

You can't use `UFUNCTION` with functions, as in, a function not inside a class.
If you want to make a native "function" callable from Blueprints, use a
static method:


{% highlight cpp %}
UCLASS()
class POWERTD_API UMathUtils : public UObject {
  GENERATED_BODY()

public:
  UFUNCTION(BlueprintCallable, Category = Math)
  static int32 GiveMeTheAnswer() { return 42; }
};
{% endhighlight %}




Structs
--------

{% highlight cpp %}
USTRUCT(Blueprintable, BlueprintType)
struct FTileIndex {
  GENERATED_USTRUCT_BODY()

  FTileIndex();
  FTileIndex(const int32 X, const int32 Y);

  UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Math)
  int32 X;
  UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Math)
  int32 Y;
};
{% endhighlight %}


Numeric Limits
--------------

{% highlight cpp %}
using T = float;
TNumericLimits<T>::Lowest();
TNumericLimits<T>::Max();
TNumericLimits<T>::Min();
{% endhighlight %}

Blueprint Node With More Than One Output Pin
----------------------

By using parameters as references we can create Blueprint
nodes with many output pins.

In the following example, `Destiny` is an input pin while
the rest of the bools are all output pins.

{% highlight cpp %}
class ... {
  UFUNCTION(BlueprintCallable, Category = EnergyStorage)
  void CreateOutputLink(UACEnergyStorage *Destiny, bool &bSuccess,
                        bool &bTargetInvalid, bool &bLinkAlreadyExists,
                        bool &bCycleDetected, bool &bMaxLinksReached, bool &bOutOfRange);
}
{% endhighlight %}


UObject::PostInitProperties()
-----------------------------

Very handy method from `UObject` that will be called after the
properties has been initialized. This include stuff set in
Blueprints :)

{% highlight cpp %}
// in .h
UCLASS(Abstract, Blueprintable, BlueprintType)
class POWERTD_API UBaseTowerData : public UObject {
  GENERATED_BODY()

public:

  void PostInitProperties() override;
}

// in .cpp
void UBaseTowerData::PostInitProperties() {
  Super::PostInitProperties(); // Required call to super
  // Do your computation here
}
{% endhighlight %}


TSubclassOf
-----------

Let's say you have to spawn an `UObject` from C++, but this
object is a Blueprint type specializing a C++ type.
One good way of doing it is by asking for its class on
a property with `TSubclassOf`, then the class can be set
in the Editor.

`TSubclassOf` is a handy little class that filters the class
selection on a property. This means only classes derived from
the type passed to it will be shown in the Editor.

Here is an example:

{% highlight cpp %}
// in .h
class ... {
  UPROPERTY(EditDefaultsOnly, Category = TowerInfo)
  TSubclassOf<UBaseFOR> DefaultFOR;
  // In the editor, only classes derived from `UBaseFOR`
  // will be shown.
}

// in .cpp
void ...() {
  // Constructing our Blueprint type here :)
  if (*DefaultFOR) {
    auto *FOR = NewObject<UBaseFOR>(this, *DefaultFOR);
    this->PlugFOR(FOR);
  }
  else {
	  UE_LOG(BaseTowerData, Error, TEXT("Default Initial FOR Not Specified"));
  }
}
{% endhighlight %}

Full details [here](https://docs.unrealengine.com/latest/INT/Programming/UnrealArchitecture/TSubclassOf/index.html).


Expose Properties On Spawn
--------------------------

In Blueprints we have "Expose On Spawn" on properties, how about C++? Yeap,
we have it too :)

{% highlight cpp %}
UCLASS(Abstract, Blueprintable, Category = EnergyTransfer)
class POWERTD_API AEnergyTransfer : public AActor {
  GENERATED_BODY()

public:
  UPROPERTY(BlueprintReadWrite, EditAnywhere, Meta = (ExposeOnSpawn = true),
            Category = EnergyTransfer)
  FVector StartLocation;
  UPROPERTY(BlueprintReadWrite, EditAnywhere, Meta = (ExposeOnSpawn = true),
            Category = EnergyTransfer)
  FVector EndLocation;
  UPROPERTY(BlueprintReadWrite, EditAnywhere,
            Category = EnergyTransfer)
  FVector PointsOffset;
};
{% endhighlight %}


Spawning Actors with Expose-On-Spawn Properties
-----------------------

But, how about spawning Actors with Expose-On-Spawn properties?

{% highlight cpp %}

  // SpawnActorDeferred is the magic here
  // "this::TransferActorClass" is set in Blueprint with TSubclassOf, see above.
  auto *EnergyTransfer = GetWorld()->SpawnActorDeferred<AEnergyTransfer>(
      /* UClass* */ *this->TransferActorClass,
      /* FTransform const& */ FTransform()
      /* AActor* Owner, APawn* Instigator, bool bNoCollisionFail */); // optionals
  if (EnergyTransfer == nullptr) {
	  return;
  }
  if (this->GetOwner() == nullptr) {
	  return;
  }

  // Use this space to set any property of your actor :)
  // I'm setting the two properties from the example above
  EnergyTransfer->StartLocation = this->GetOwner()->GetActorLocation();
  EnergyTransfer->EndLocation = Destiny->GetOwner()->GetActorLocation();

  // Then finish spawning it...
  UGameplayStatics::FinishSpawningActor(EnergyTransfer, FTransform());
{% endhighlight %}



Spawning Particle Systems
----------------------

At location:

{% highlight cpp %}
  // "this::EnergyPS" is of type "UParticleSystem" and is set in the Editor.
  UParticleSystemComponent* PS = UGameplayStatics::SpawnEmitterAtLocation(
      GetWorld(), /* UParticleSystem EmitterTemplate */ this->EnergyPS,
      /* FVector location */ this->StartLocation
      /* FRotator rotation,  bool bAutoDestroy */); // These are optional
{% endhighlight %}



Delegates and Blueprints
---------------------

Need more research.



 [1]: http://llvm.org/builds/
 [2]: /assets/images/vs_slow.png
 [3]: https://github.com/syl20bnr/spacemacs/
 [4]: https://docs.unrealengine.com/latest/INT/Programming/UnrealArchitecture/Reference/Properties/Specifiers/index.html 
 [5]: https://docs.unrealengine.com/latest/INT/Programming/UnrealArchitecture/Reference/Functions/Specifiers/index.html
 [6]: https://docs.unrealengine.com/latest/INT/Programming/UnrealArchitecture/Reference/Classes/Specifiers/index.html 
